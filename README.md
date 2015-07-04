# sshdeploy
 [![Analytics](https://ga-beacon.appspot.com/UA-8535255-2/unosquare/sshdeploy/)](https://github.com/igrigorik/ga-beacon)

A command-line tool that enables quick deployments over SSH. This is program was specifically designed to streamline .NET application development for the the Raspberry Pi running Raspbian. 

<h2>Usage Example</h2>

The following steps outline a continuous deployment of a Visual Studio solution to a Raspberry Pi running the default Raspbian SSH daemon.

<ol>
     <li>Download and compile this tool. You should end up with SshDeploy.exe and a few dll files.</li>
     <li>Open a Command Prompt (Start, Run, cmd, [Enter Key])</li>
     <li>
     Navigate to the folder where the SshDeploy.exe file is located (output of step 1).
     <br />Example:
     <code>cd "C:\projects\sshdeploy\trunk\Unosquare.Labs.SshDeploy\bin\Debug"</code>
     </li>
     <li>Now go to your Visual Studio Solution (the one you intend to continously deploy to the Raspberry Pi).
     </li>
     <li>Right click on the project and click on the menu item "Properties"</li>
     <li>Go to the "Build Events" tab, and under Post-build events, enter the following: <br />
         <code>
         echo %DATE% %TIME% >> "$(TargetDir)sshdeploy.ready"
         </code>
         <br />
         This simply writes the date and time to the <code>sshdeploy.ready</code> file. Whenever this file CHANGES, the deployment tool will perform a deployment.
     </li>
     <li>
     Go back to the Command Prompt window you opened in a prior step and run this tool with some arguments. Here is an example so you can get started quickly.
     <br />
     <code>sshdeploy monitor -s "C:\projects\Unosquare.Labs.RasPiConsole\Unosquare.Labs.RasPiConsole\bin\Debug" -t "/home/pi/target" -h 192.168.2.194 -u pi -w raspberry
     </code>
     <br />
     In the above command,
     <ul>
     <li><code>-s</code> refers to the full path of the source folder.</li>
     <li><code>-t</code> refers to the full path of the target directory.</li>
     <li><code>-h</code> refers to the host (IP address of the Raspberry Pi).</li>
     <li><code>-u</code> refers to the login.</li>
     <li><code>-w</code> refers to the password.</li>
     </ul>
     Note that there are many more arguments you can use. Simply issue <br />
     <code>
     sshdeploy monitor
     </code>
     <br />This will get you all the options you can use.
     </li>
     <li>If all goes well you will see output similar to this:<br />
     <pre>
SSH Deployment Tool [Version 1.0.0.0]
(c) 2015 Unosquare SA de CV. All Rights Reserved.

Monitor mode starting
Monitor parameters follow:
    Monitor File    C:\projects\Unosquare.Labs.RasPiConsole\Unosquare.Labs.RasPiConsole\bin\Debug\sshdeploy.ready
    Source Path     C:\projects\Unosquare.Labs.RasPiConsole\Unosquare.Labs.RasPiConsole\bin\Debug
    Excluded Files  .ready|.vshost.exe|.vshost.exe.config
    Target Address  192.168.2.194:22
    Username        pi
    Target Path     /home/pi/target
    Clean Target    YES
    Pre Deployment
    Post Deployment
Connecting to host 192.168.2.194:22 via SFTP.
File System Monitor is now running.
Writing a new monitor file will trigger a new deployment.
Remember: Press Q to quit.
Ground Control to Major Tom: Have a nice trip in space!
     </pre>
     </li>
     <li>Now go back to your Visual Studio Solution, right click on the project, a select "Rebuild". You should see output in the command line similar to the following:<br />
     <pre>
     Starting deployment ID 1 - Sunday, June 14, 2015 10:16:20 PM
     Cleaning Target Path '/home/pi/target'
     Deploying 3 files.
     Finished deployment in 0.88 seconds.
     </pre>
     </li>
     <li>
     Every time you rebuild your project, it will be automatically deployed!
     </li>
</ol>

<i>In order to make this tool much more useful, we need to take advantage of the pre and post commands. The idea is to find the process and kill it if it is currently running on the pre-command, and run the process once the deployment has been completed using the post-command argument. The hope is thay this will make the deploy, run, and debug cycle, much less tedious for a .NET developer using a Raspberry Pi.</i>

Here's a good example of using pre and post commands to acocmplish the above

<code>monitor -s "C:\projects\libfprint-cs\trunk\Unosquare.Labs.LibFprint.Tests\bin\Debug" -t "/home/pi/libfprint-cs" -h 192.168.2.194 --pre "pgrep -f 'Unosquare.Labs.LibFprint.Tests.exe' | xargs -r kill" --post "mono /home/pi/libfprint-cs/Unosquare.Labs.LibFprint.Tests.exe" --clean False
</code>

<h2>Monitor Mode Documentation</h2><small>from command line help output</small>

<pre>
  -m, --monitor     (Default: sshdeploy.ready) The path to the file used as a
                    signal that the files are ready to be deployed. Once the
                    deployemtn is completed, the file is deleted.

  -s, --source      Required. The source path for the files to transfer

  -t, --target      Required. The target path of the files to transfer

  --pre             Command to execute prior file transfer to target

  --post            Command to execute after file transfer to target

  --clean           (Default: True) Deletes all files and folders on the target
                    before pushing the new files

  --exclude         (Default: .ready|.vshost.exe|.vshost.exe.config) a pipe (|)
                    separated list of file suffixes to ignore while deploying.

  -v, --verbose     (Default: True) Add this option to print messages to
                    standard error and standard output streams.

  -h, --host        Required. Hostname or IP Address of the target. -- Must be
                    running an SSH server.

  -p, --port        (Default: 22) Port on which SSH is running.

  -u, --username    (Default: pi) The username under which the connection will
                    be established.

  -w, --password    (Default: raspberry) The password for the given username.
</pre>

<h2>Special Thanks</h2>
<p>
This code uses the very cool Renci's <a href="https://sshnet.codeplex.com/" target="_blank">SSH.NET library</a> and the awesome Command Line Parser library available <a href="https://github.com/gsscoder/commandline" target="_blank">here</a>.
</p>