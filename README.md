Download Link: https://assignmentchef.com/product/solved-project-1-system-calls-and-processes
<br>
<h1>Introduction</h1>

The purpose of this project is to study how system calls and processes are managed by the Linux OS. The objectives of this project is to:

<ol>

 <li>Get familiar with the Linux environment.</li>

 <li>Learn how to build a Linux kernel.</li>

 <li>Learn how to add a new system call in the Linux kernel.</li>

 <li>Learn how to obtain various information for a running process from the Linux kernel.</li>

</ol>

<h1>Project submission</h1>

For each project, please create a zipped file containing the following items, and submit it to Canvas.

<ol>

 <li>A report that includes (1) the (printed) full names of the project members, and the statement: <strong>We have neither given nor received unauthorized assistance on this work</strong>; (2) the name of your virtual machine (VM), and the password for the instructor account (details see <em>Accessing VM </em>below); and (3) a brief description about how you solved the problems and what you learned. For example, you may want to make a typescript of the steps in building the Linux kernel, or the changes you made in the Linux kernel source for adding a new system call, or screenshots of your result. The report can be in txt, doc, or pdf format.</li>

 <li>The source code including both the kernel-level code you added and the user-level testing program.<a href="#_ftn1" name="_ftnref1"><sup>[1]</sup></a></li>

</ol>

<em>Accessing VM. </em>Each team should specify the name of your VM that the instructor can login to check the project results. Please create a new user account called instructor in your VM, and place your code in the home directory of the instructor account (i.e., /home/instructor). Make sure the instructor has the appropriate access permission to your code. In your project report, include your password for the instructor account.

<h1>Tasks</h1>

<h2>Task 0: Building the Linux kernel (20 points)</h2>

<h3>Step 1: Getting the Linux kernel code</h3>

Before you download and compile the Linux kernel source, make sure that you have development tools installed on your system.

In CentOS, install these software using yum (Note that the # prompt indicates that the command requires root privilege. $ indicates normal user privilege):

# yum install -y gcc ncurses-devel make wget

In the past some VMs don’t have perl pre-installed. If that’s the case, do yum install perl with root privilege.

Visit http://kernel.org and download the source code of your current running kernel. To obtain the version of your current kernel, type:

<table width="576">

 <tbody>

  <tr>

   <td width="576">$ uname -r</td>

  </tr>

  <tr>

   <td width="576">$ 2.6.32-220.el6.i686</td>

  </tr>

 </tbody>

</table>

Then, download kernel 2.6.32 and extract the source:

<table width="576">

 <tbody>

  <tr>

   <td width="576">$ wget http://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.32.tar.gz</td>

  </tr>

  <tr>

   <td width="576">$ tar xvzf linux-2.6.32.tar.gz</td>

  </tr>

 </tbody>

</table>

We will use LINUX SOURCE to refer to the top directory of the kernel source, i.e., linux-2.6.32/.

<h3>Step 2: Configure your new kernel</h3>

<strong>It’s a good idea to take a snapshot of your VM in case things get messed up. However, please keep the number of snapshots under 4</strong>.

Before compiling the new kernel, a .config file needs to be generated in the top directory of the kernel source. To generate the config file, type this <strong>in </strong>LINUX SOURCE:

$ make localmodconfig

<strong>Make sure that you hit “enter” when prompted for questions.</strong>

<h3>Step 3: Compile the kernel</h3>

<strong>In </strong>LINUX SOURCE, compile to create a compressed kernel image:

$ make -j8

Replace 8 in make -j8 with the number of cores you have for the VM (typically a number between 4-8. if you don’t know, this wouldn’t affect you, just an optimization option).

<h3>Step 4: Install the kernel</h3>

Compile and install kernel modules (to become the root user, use the su command):

$ su

<table width="576">

 <tbody>

  <tr>

   <td width="576"># make modules</td>

  </tr>

  <tr>

   <td width="576"># make modules install</td>

  </tr>

 </tbody>

</table>

Again, in the above, $ means you are a normal user, and # means you are root. Now install the kernel:

<table width="576">

 <tbody>

  <tr>

   <td width="576"># export KERNELVERSION=$(make kernelversion)</td>

  </tr>

  <tr>

   <td width="576"># cp arch/x86/boot/bzImage /boot/vmlinuz-$KERNELVERSION</td>

  </tr>

  <tr>

   <td width="576"># cp System.map /boot/System.map-$KERNELVERSION</td>

  </tr>

 </tbody>

</table>

<strong>When you run the two </strong>cp <strong>commands above, and if you are asked whether to overwrite the existing files, make sure that you hit </strong>y <strong>to overwirte. </strong>The first time you ran it, you wouldn’t see this message.

# new-kernel-pkg –mkinitrd –install $KERNELVERSION

The kernel image and other related files have been installed into the /boot directory.

<h3>Step 5: Reboot your VM</h3>

Reboot to the new kernel:

# reboot

Please make sure that you select the new kernel that has been built (i.e., 2.6.32).<a href="#_ftn2" name="_ftnref2"><sup>[2]</sup></a> After reboot, check if you have the new kernel:

<table width="576">

 <tbody>

  <tr>

   <td width="576">$ uname -r</td>

  </tr>

  <tr>

   <td width="576">$ 2.6.32</td>

  </tr>

 </tbody>

</table>

<h2>Task 1: Add a new system call to the Linux kernel (20 points)</h2>

In this task, we will add a simple system call helloworld to the Linux kernel. The system call prints out a Hello World! message to the syslog. You need to implement the system call in the kernel and write a user-level program to test your new system call.

<h3>Step 1: Implement your system call</h3>

Change your current working directory to the kernel source directory.

$ cd LINUX SOURCE

Make a new directory called my source to contain your implementation:

$ mkdir my source

Create a C file and implement your system call here:

<table width="576">

 <tbody>

  <tr>

   <td width="576">$ touch my source/sys helloworld.c</td>

  </tr>

 </tbody>

</table>

Edit the source code to include the following implementation (be careful about the quotation marks – type the printk function by hand rather than copy&amp;paste to avoid encoding issues):

<table width="576">

 <tbody>

  <tr>

   <td width="576">$ vim my source/sys helloworld.c</td>

  </tr>

 </tbody>

</table>

#include &lt;linux/kernel.h&gt; #include &lt;linux/sched.h&gt; asmlinkage int sys helloworld (void)

{

printk(KERN EMERG ‘‘Hello World!
’’); return 0;

}

Add a Makefile to the my source folder:

<table width="576">

 <tbody>

  <tr>

   <td width="576">$ touch my source/Makefile</td>

  </tr>

  <tr>

   <td width="576">$ vim my source/Makefile</td>

  </tr>

 </tbody>

</table>

#

#Makefile of the new system call

#

obj-y := sys helloworld.o

Modify the Makefile in the top directory LINUX SOURCE to include the new system call in the kernel compilation:

$ vim Makefile

Find the line where core-y is defined and add the my source directory to it:

core-y += kernel/ mm/ fs/ ipc/ security/ crypto/ block/ my source/

Add a function pointer to the new system call in arch/x86/kernel/syscall table 32.S:

<table width="576">

 <tbody>

  <tr>

   <td width="576">$ vim arch/x86/kernel/syscall table 32.S</td>

  </tr>

 </tbody>

</table>

Add the following line to the end of this file:

.long sys helloworld /* 337 */

Now you have defined helloworld as system call 337.

Register your system call with the kernel by editing arch/x86/include/asm/unistd 32.h. If the last system call in arch/x86/kernel/syscall table 32.S has an id number 336, then our new system call should be numbered as 337. Add this to the file unistd 32.h, right below the definition of system call 336:

<table width="576">

 <tbody>

  <tr>

   <td width="76">#define</td>

   <td width="500">NR helloworld 337</td>

  </tr>

 </tbody>

</table>

Modify the total number of system calls to 337 + 1 = 338:

#define NR syscalls 338

Declare the system call routine. Add the following to the end of arch/x86/include/asm/syscall.h (right before the line CONFIG X86 32, near the end of the file):

<table width="576">

 <tbody>

  <tr>

   <td width="576">asmlinkage int sys helloworld(void);</td>

  </tr>

 </tbody>

</table>

<strong>Repeat steps 3, 4 and 5 in task 0 to re-comple, install and reboot to the new kernel.</strong>

<h3>Step 2: Write a user-level program to test your system call</h3>

Go to your home directory and create a test program test syscall.c:

#include &lt;linux/unistd.h&gt;

#include &lt;sys/syscall.h&gt;

#include &lt;sys/types.h&gt;

#include &lt;stdio.h&gt; #define NR helloworld 337 int main(int argc, char *argv[]) { syscall( NR helloworld); return 0;

}

Compile the program:

$ gcc test syscall.c -o test syscall

Test the new system call by running:

<table width="576">

 <tbody>

  <tr>

   <td width="576">$ ./test syscall</td>

  </tr>

 </tbody>

</table>

If your program hangs, hit <strong>enter </strong>to continue. The test program will call the new system call and output a helloworld message at the tail of the output of dmesg (try dmesg | tail to see the output).

<h2>Task 2: Extend your new system call to print the calling process’s information (30 points)</h2>

Follow the instructions we discussed above and implement another system call print self. This system call identifies the current process at the user-level and print out various information of the process.

Implement the print self system call and print out the following information of the current process:

<ul>

 <li>Process id and program name</li>

 <li>The same above information of its parent processes until init</li>

</ul>

HINT: The macro current returns a pointer to the task struct of the current running process. See the following link: http://linuxgazette.net/133/saha.html Please pay attention to the last example that traces from the current process back to init.

<h2>Task 3: Extend your new system call to print the information of an arbitrary process identified by its PID (30 points)</h2>

Implement another system call print other to print the information for an arbitrary process. The system call takes a process pid as its argument and outputs the above information (same as Task 2) of this process.

HINT: You can start from the init process and iterate over all the processes. For each process, compare its pid with the target pid. If there is a match, return the pointer to this task struct. A better approach would be to use the functions that Linux provides to look up the process in the process table. You may refer to the lecture slides.

To get a valid PID for a process that’s active, you may use command pgrep bash. bash is the default command line shell. pgrep bash will provide PIDs of bash. You can choose any of them if you get more than one. More information about command line tools to get process information:

http://linuxcommando.blogspot.com/2007/11/how-to-look-up-process.html

<a href="#_ftnref1" name="_ftn1">[1]</a> In case there are issues accessing your VM, submitting your source code separately would allow the instructor to give partial credit.

<a href="#_ftnref2" name="_ftn2">[2]</a> In the past, some people had the issue of not having an option to select a new build upon reboot. If this happens, try holding down the shift key while reboot. <strong>When you see a message “Boot CentOS (2.6.32-220.el6.i686)” hit enter right away. </strong>Note that if you use ssh, a reboot would disconnect your laptop with the VM. So you may use the GUI interface to reboot.