## Configure IIS and ASP.NET
__Enable IIS and ASP.NET__  
* Windows Start > Control Panel
* Program > Turn Windows features on or off
* Before you begin > Next
* Installation type = "Role-based or feature based installation"  > Next  
* Server Selection = "Select a server from the pool"  > Next
* Server Roles =  

```
* Web Server (IIS)
   * Web Server (all)
     * Application Development (Most)
       * ASP.NET 4.5
       ... Others Stuff
   * Management Tools    
```
* Click Next    
* Features = .NET Framework 4.5 Features > Next  
* Confirmation
* Install  

If you are working on Windows 10 IIS in your development environment, make sure to check all the option under `Application Development Feature` (maybe except CGI)
```
* Internet Information Services
  * World Wide Web Services
    * Application Development
      * .NET Extensibility 3.5
      * .NET Extensibility 4.8
      ... Others Stuffs

```

__Create Website and then add entry to host file__  
Open host file `C:\Windows\System32\drivers\etc\hosts` and add entry.
```
127.0.0.1 example.com
```
The host name of your website in this case is `example.com` and must be the host name in your site Bindings on IIS.  
