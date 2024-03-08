<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Exploring Network Security Groups (NSGs) and Analyzing Traffic Between Azure Virtual Machines</h1>
<li>This repository provides guidance on inspecting network traffic to and from Azure Virtual Machines using Wireshark and experimenting with Network Security Groups. It builds upon the foundation laid in the <a href="https://github.com/jadeblas/configure-ad">active directory configuration repository</a>, assuming that you have completed the setup outlined there.</li>

<h2>Environments and Technologies Used</h2>
<ul>
  <li>Microsoft Azure (Virtual Machines/Compute)</li>
  <li>Remote Desktop</li>
  <li>Various Command Line Tools</li>
  <li>Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)</li>
  <li>Wireshark (Protocol Analyzer)</li>
</ul>

<br />

<h2>Operating Systems Used</h2>
<ul>
  <li>Windows Server 2022</li>
  <li>Windows 10 Pro (21H2)</li>
  <li>Ubuntu Server 20.04</li>
</ul>

<br />

<h2>List of Prerequisites</h2>
<ol>
  <li>Microsoft Azure Account and Subscription</li>
  <li>Access to Microsoft Remote Desktop Connection</li>
  <ul>
    <li>For MacOS users, follow <a href = "https://www.youtube.com/watch?v=0lllpAhgAJs&ab_channel=TheHostingVideos">this video</a> to use Remote Desktop on Mac</li>
  </ul>
</ol>

<h2>Steps</h2>

<h3>Setting Up Sample Fileshares with Varied Permissions</h3>

<p>
  <ul>
    <li>Ensure that your Domain Controller VM is connected and logged in as an administrator (<b>mydomain.com\jane_admin</b>). Similarly, ensure that your Client VM is connected and logged in as a user generated through the PowerShell script during the configuration process.</li>
    <li>Within the Domain Controller VM, navigate to the C:\ Drive and create the following four folders, each with specific permissions:</li>
    <ul>
      <li><b>read_access</b>: Grant the Domain Users group Read permissions.</li>
      <li><b>write_access</b>: Grant the Domain Users group Read/Write permissions.</li>
      <li><b>no_access</b>: Grant the Domain Admins group Read/Write permissions.</li>
      <li><b>accounting</b>: This folder is reserved for future use and will be addressed later.</li>
    </ul>
    <li>Example of setting group and permissions for the <b>read_access</b> folder:</li>
    <ul>
     <li><img src ="https://github.com/jadeblas/azure-network-protocols/assets/161860082/8d6322a7-19ad-438b-b8e9-2add8c667922" width = 80% height = 80%/></li>
    </ul>
  </ul>
</p>

<br/>

<h3>Attempting Fileshare Access</h3>

<p>
  <ul>
    <li>On the Client VM, open <b>File Explorer</b> and navigate to the shared folder by entering <b>\\dc-1</b> in the address bar.</li>
    <ul>
      <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/d244996d-57cd-4234-b931-c025ceed0a67" width="80%" height="80%" alt="Accessing Shared Folder via File Explorer"/></li>
    </ul>
    <li>Attempt to access the folders from the Client VM. Based on the permissions set, only the <b>no_access</b> folder should be inaccessible.</li>
    <ul>
      <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/2cd159e9-9aef-47a5-aea3-9b7949f7e0f1" width="80%" height="80%" alt="Attempting to Access Folders"/></li>
    </ul>
  </ul>
</p>

<br/>

<h3>Establishing the "ACCOUNTANTS" Security Group</h3>

<p>
  <ul>
    <li>Return to the Domain Controller VM and access the Server Manager Dashboard. Navigate to <b>Active Directory Users and Computers</b> to create a new <b>Organizational Unit (OU)</b> named "_SECURITY_GROUP."</li>
    <ul>
      <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/b67d4301-803b-4105-a401-e29661e06f4d" width="80%" height="80%" alt="Creating OU"/></li>
    </ul>
    <li>Within the "_SECURITY_GROUP" OU, create a new <b>Group</b> named "ACCOUNTANTS."</li>
    <ul>
      <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/42b6595f-fe58-4901-9620-ef93d0d421dc" width="80%" height="80%" alt="Creating Security Group"/></li>
    </ul>
    <li>Locate the previously created <b>accounting</b> folder on the C:\ Drive and add the "ACCOUNTANTS" group, granting it Read/Write permissions.</li>
    <ul>
      <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/00195ef6-e76b-4b56-ad9b-aac1951ae07f" width="80%" height="80%" alt="Setting Permissions for ACCOUNTANTS"/></li>
    </ul>
    <li>As the user logged into the Client VM does not belong to the "ACCOUNTANTS" group, they should not have access to the accounting folder. Log out of the Client VM and make a note of the username used to log in, as it will be added to the "ACCOUNTANTS" group.</li>
    <li>In the Domain Controller VM, navigate to the "_SECURITY_GROUP" OU, right-click on the "ACCOUNTANTS" group, select <b>Properties</b>, go to the <b>Members</b> tab, and add the user as a member of the group.</li>
    <ul>
      <li>In this example, the user "ban.doh" is used.</li>
      <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/cb67ca66-436a-401e-9bbf-2b1ee621422f" width="80%" height="80%" alt="Adding User to ACCOUNTANTS Group"/></li>
    </ul>
    <li>Sign back into the Client VM with the user added to the "ACCOUNTANTS" group, and they should now have access to the accounting folder.</li>
  </ul>
</p>

<br/>

<h2>Observing Traffic in Virtual Machines</h2>

<h3>Download and Install Wireshark</h3>

<p>
  <ul>
    <li>First, download <a href="https://www.wireshark.org/download.html">Wireshark</a> in your VM. Downloads may be slow depending on your VM's CPU</li>
  </ul>
</p>

<br />

<h3>Examining ICMP (Internet Control Message Protocol) Traffic</h3>

<p>
  <ul>
    <li>After installation, launch Wireshark and initiate packet capture by clicking on the blue fin icon. In the filter bar, type <b>icmp</b> to focus on incoming ICMP packets.</li>
    <ul>
      <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/ccc41d87-c68e-4965-8a89-94d2432ace84" height="80%" width="80%" alt="Wireshark Capture Setup"/></li>
    </ul>
    <li>Switching to your physical desktop, access your Microsoft Azure Account to obtain the <b>Private IP Address</b> of VM-2 and make a note of it.</li>
    <ul>
    <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/2e2829ff-0a9f-480b-9bf1-d77b8484c669" height="50%" width="50%" alt="Obtaining Private IP Address"/></li>
    </ul>
    <li>In VM-1's Windows Powershell, execute the command <b>ping</b> followed by the private IP of VM-2. This action should generate ICMP packets visible in Wireshark.</li>
    <ul>
    <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/2e508909-48f9-4303-8586-6fdbcbd23d98" height="80%" width="80%" alt="Executing Ping Command"/></li>
    </ul>
    <li>To establish a continuous ping between the Virtual Machines, execute the <b>ping</b> command followed by the private IP of VM-2 and add <b>-t</b>, causing uninterrupted ICMP packet transmission visible in Wireshark.</li>
    <ul>
    <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/0c29fcad-de16-49e7-91fb-c92e763b16e7" height="80%" width="80%" alt="Continuous Ping Setup"/></li>
    </ul>
    <li>Returning to the Microsoft Azure Account, navigate to VM-2's <b>Network Security Group (NSG)</b> (typically named <i>VM-2-nsg</i>) to interrupt the traffic.</li>
    <li>Within VM-2-nsg, access <b>inbound security rules</b> and create a rule denying ICMP traffic. Click on <b>Add</b>, set <b>Deny</b> under action, <b>ICMP</b> under Protocol, prioritize above 300, and name the rule <b>DENY_ICMP_PING</b>, then click <b>Add</b> to finalize.</li>
    <ul>
    <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/b83fec7f-1aae-485d-8f75-ee7ce4a2bee9" height="80%" width="80%" alt="NSG Rule Configuration"/></li>
    </ul>
    <li>Upon completion, observe the "Request timed out" message in VM-1's Powershell, indicating the halt of ICMP ping due to the security rule.</li>
    <ul>
    <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/b7196621-1d68-4dae-b711-1b0719428e36" height="80%" width="80%" alt="ICMP Traffic Halted"/></li>
    </ul>
    <li>To restore traffic, return to your Microsoft Azure Account and set the action of the DENY_ICMP_PING inbound rule to <b>Allow</b>, then save the changes.</li>
  </ul>
</p>

<br />

<h3>Monitoring SSH (Secure Shell) Traffic</h3>

<p>
<ul>
  <li>Within VM-1's Windows Powershell, enter <b>ssh VM-2@[VM-2's Private IP]</b> and press Enter. Confirm the connection by typing "yes" when prompted, and then provide the password for VM-2.</li>
  <li>As the Terminal of VM-2 (similar to Linux's command prompt) is accessed, password input doesn't display dots, but rest assured that input is being registered.</li>
  <li>Upon successful login, you will be connected to the Terminal of VM-2. To exit, simply enter the command <b>exit</b>.</li>
  <ul>
  <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/9c801bb7-5758-4109-aa45-70b0744a0e31" height="80%" width="80%" alt="SSH Connection Steps"/></li>
  </ul>
  <li>Executing commands such as <i>username</i>, <i>pwd</i>, or <i>sudo apt</i> will generate traffic visible in Wireshark. To focus on SSH traffic, use the filter bar and type <b>ssh</b>.</li>
</ul>
</p>

<br />

<h3>Observing DHCP (Dynamic Host Configuration Protocol) Traffic</h3>

<p>
  <ul>Filter DHCP Traffic in Wireshark by entering <b>dhcp</b> in the filter bar.</ul>
  <ul>DHCP dynamically assigns IP Addresses to devices joining the network. To reassign an IP Address in the VM, access Powershell and execute the command <b>ipconfig /renew</b>.</ul>
</p>

<br/>

<h3>Examining DNS (Domain Name System) Traffic</h3>

<p>
  <ul>
    <li>Utilize Wireshark's filter bar and enter <b>dns</b> to focus on DNS traffic.</li>
    <li>In Powershell, execute the command <b>nslookup</b> followed by a website such as google.com to observe DNS resolution.</li>
  </ul>
</p>

<br/>

<h3>Monitoring RDP (Remote Desktop Protocol) Traffic</h3>

<p>
  <ul>
    <li>Apply a filter in Wireshark by entering <b>tcp.port == 3389</b> to isolate RDP traffic, where you'll notice continuous transmission.</li>
    <li>RDP facilitates a live stream between computers, resulting in constant traffic transmission.</li>
  </ul>
</p>

<br/>

<h2>Cleaning Up</h2>

<ul>
  <li>Logout from Remote Desktop Connection.</li>
  <li>It is recommended to delete the Resource Group and VMs once experimentation is complete to avoid future costs. Deletion of assets on Azure requires verification by entering the name of the asset. Note that the Resource Group <b>NetworkWatcherRG</b> is created when creating NSGs for Virtual Machines and necessitates separate deletion.</li>
  <ul>
  <li><img src="https://github.com/jadeblas/azure-network-protocols/assets/161860082/044947d6-bde6-47e8-8643-c966e1479bc2" height="80%" width="80%" alt="Cleanup Steps"/></li>
  </ul>
</ul>
