<h1>Enumeration</h1>
<p>We start by pinging the target IP 10.10.11.20</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/08362971-cead-4896-a56e-fe40e75960f0" alt="Ping to target IP">
</center>
<br>

<p>Next, we perform a simple nmap scan to determine which ports are open on the target machine:</p>
<pre><code>sudo nmap -p- --open -sV -sS -n 10.10.11.20 --min-rate 5000</code></pre>
<p>This reports the following open ports:</p>
<ul>
    <li>Port 22 running OpenSSH</li>
    <li>Port 80 running Nginx web server (http)</li>
</ul>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/4b2effdb-f0b9-40c9-a839-c58bf90c92d9" alt="Nmap scan results">
</center>
<br>

<h1>Checking the Web Server</h1>
<p>Let's open the browser and check the web server:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/9acff6bd-4f3d-4575-9021-a44b7bf6c1f4" alt="Checking the web server">
</center>
<br>

<p>We can see the target IP resolves its name on editorial.htb, so we add this to our local /etc/hosts file for quicker resolution:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/02037fad-5f58-4e75-9aae-173b4b99cb8b" alt="Updating /etc/hosts">
</center>
<br>

<h1>Exploitation</h1>
<p>Reloading the server, we can see the website editorial.htb. Let's explore to find something interesting...</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/587b4d5d-c407-4c62-86ba-5c8316af0c06" alt="Website editorial.htb">
</center>
<br>

<p>There is a directory editorial.htb/upload that allows us to upload URLs and images. We use Burp Suite to inspect how the server handles this request. If we input a URL in the book URL field and send the request using Burp Suite Repeater, the server responds with a 200 OK status, indicating an SSRF vulnerability.</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/94fc8347-b13b-4f78-87db-834ab0d47af9" alt="Burp Suite - SSRF Vulnerability">
</center>
<br>

<p>Knowing this security gap, we can try sending the local IP (127.0.0.1) on port 5000 (a common port for SSRF exploits). Alternatively, we can check all ports (1-65535) for unusual server responses. To demonstrate, we send the request to Burp Suite Intruder and launch a payload to check the server's response on different ports:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/4aa1d854-2fc9-4241-a09c-b495bb0eb21d" alt="Burp Suite Intruder - Checking Ports">
</center>
<br>

<p>Let's try this list of ports: 1, 2, 5, 6, 80, 443, 5000, 8080</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/63a2620a-bc9f-4dd4-8881-5ce8121aed9c" alt="Port List">
</center>
<br>

<p>Port 5000 gives the shortest length (222), suggesting a previous directory. Let's check this path with Burp Suite Repeater:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/a5e3c986-c081-4fcf-9912-81a013a68e9c" alt="Burp Suite Repeater - Directory Path">
</center>
<br>

<p>We try to get the content of this path. If we use a POST request, we get an encrypted response, so we try a GET request:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/33c8ccc8-6923-4107-9d23-c0e32ade964a" alt="GET Request">
</center>
<br>

<p>In the static/uploads directory, we find a JSON file containing information about various APIs used by the web server. The /api/latest/metadata/messages/authors API catches our attention. We repeat the same process with this path:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/cdde3c9f-dae9-4d34-b583-34b2bd2e46b5" alt="JSON File - API Information">
</center>
<br>

<p>Finally, we obtain a response with sensitive information about this API data. We use these credentials to start an SSH session:</p>
<br>
<center>
    <img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*-lbTLf929CaUAHaWWoLQbQ.png">
</center>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/cde3b765-7f5a-4afd-a658-cf89af3f7d1d" alt="SSH Login">
</center>
<br>

<p>We can log in to SSH with the obtained credentials. We capture the user's flag by running <code>cat user.txt</code>. First objective completed, now let's get the root flag.</p>

<p>We navigate to the dev user's directories and find a .git directory. Let's see what it contains:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/a815b606-687e-4f9d-a933-84db4dbdd1d9" alt=".git Directory">
</center>
<br>

<p>There are several directories within the git repository. We check the logs directory to see if there is any information about another user with elevated permissions. The second log entry suggests a file with the creation information of the editorial API. We use the <code>git show</code> command and the log ID to view its contents:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/635ca00b-1685-4352-b021-ab04c1b8cd62" alt="git show Command">
</center>
<br>

<p>We find a new user on the system with a password. We log in to SSH with this new user:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/dd4ef326-f2bd-448d-88ca-17dd0bc666de" alt="SSH Login - New User">
</center>
<br>

<p>There is no interesting file and we are not root, so we run <code>sudo -l</code> to check what permissions we have with this user:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/9870d3b8-d78d-4ea0-b984-969c6cd83909" alt="sudo -l Command">
</center>
<br>

<p>We have root permissions to run a Python script that allows us to copy files. We exploit this script to copy the /root/root.txt directory to our /home/root.txt:</p>
<pre><code>sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py "ext::sh -c 'cat /root/root.txt > /home/prod/root.txt'"</code></pre>

<p>And we obtain the root.txt flag, completing the Editorial HTB machine. Congratulations!</p>
<br>
<img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/0406d8fe-60be-497b-8628-1e8d83b8adf8" alt="Root Flag">
</center>
