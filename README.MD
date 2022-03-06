# **NYU – CSGY-9163 Application Security - Spring22 – Lab 2**

# **Part 0: Working Repository and Submission**
## 1) Synchronize Your Repository and Acquire the Lab Material
---
Log into GitHub within any web browser and create an empty, **private** repository named ``<NetID>-appsec2``.

Now, using the Ubuntu 20.04.3 LTS course virtual machine (https://drive.google.com/file/d/1rhzwJaiFmKy8SbvQuLIP_EQBsIskDNMz/view?usp=sharing; `nyuappsec` is the password), open the command shell and execute the following commands, replacing ``<YourGitHubHandle>`` with your own GitHub username and ``<NetID>`` with your own NYU NetID:

> If you do not know how to create a personal access token to use with the command line, follow the instructions here: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token

> **⚠ Warning!** Using your personal access token directly within commands, as shown in the laboratory instructions, is not a secure practice. A threat actor in a position to view the command history of the user account on the system may abuse the token to gain unauthorized access and/or make unauthorized modifications to your GitHub account. Use an appropriately secured environment variable, an access-controlled file, or another mitigation technique if you want to avoid leaving your GitHub Personal Access Token in your command history.
```
cd ~
git clone https://github.com/NYUJRA/AppSec2.git AppSec2
cd AppSec2
git remote remove origin
git init
git remote add origin https://<YourGitHubHandle>:<YourPersonalAccessToken>@github.com/<YourGitHubHandle>/<NetID>-appsec2.git
git push -u origin main
```


You should now have a local working directory in ``~/AppSec2`` that is configured to use your remote GitHub repository at ``https://github.com/<YourGitHubHandle>/<NetID>-appsec2`` as a version control system.

## 2) What to Submit
---
Submit your AppSec2 repository along with detailed lab report, with screenshots, to describe the specific actions taken to complete each task. Enough detail should be provide that enable the recipient of your report to blindly reproduce your actions to achieve the same outcomes. Provide explanation to the observations that are interesting or surprising.

All lab tasks should be performed in the provided NYU-AppSec/CSGY9163 Ubuntu 20.04.3LTS virtual machine. Each task is required to include a minimum of 1 screenshot to prove your individual completion of the task. **Every screenshot is required to include the date and time displayed in your virtual machine; otherwise, credit for the particular task will be removed.**

For tasks involving source code or exploit code, include the important code snippets followed by explanation. Simply executing code without explanation will not be eligible for credit.

**Your report _must_ be written in markdown.** Create a folder in your AppSec2 repository, using "Report" as the new folder's name. Within `Report/`, create a sub-folder called "Artifacts" within it. Store all of your screenshots and related laboratory artifacts within the "Artifacts" sub-folder, and include **one** markdown file in the "Report" sub-folder, which will contain your documentation for both, Part 1 and Part 2, of this lab.

Your repository should now include the following file structure:

    - .github/
        - workflows/
          - <NetID>-regression.yml
    - GiftCardSite/
        - ...
    - Report/
        - Artifacts/
            - <NetID>-screenshot1.jpg
            - <NetID>-screenshot2.jpg
        - <NetID>-AppSec-Lab2.md
    - <NetID>-xss.py
    - <NetID>-csrf.py
    - <NetID>-sqli.py
    - <NetID>-cmdi.py

> **Note.** The GiftCardSite/ source files should be updated with all of the fixes and modifications made throughout the lab. Throughout grading, the updated source files will be reviewed and corroborated with your lab report.

## 3) How to Submit
---

You must submit your assigment in two places.
1. In BrightSpace, submit a link to your assignment repository.
2. Update the "Assignment 1 Report URL" column in the grading allocation spreadsheet (https://docs.google.com/spreadsheets/d/1y57x4X4nI1FvESEGO9lxLw0HgQtfOn0z5aO3yIqoCy8/edit?usp=sharing) with a link to your private AppSec 1 repository. 

    2a. You must invite the professor(s) to collaborate on your private repository
        \
         - Professor Allen's GitHub handle is `NYUJRA`
        \
         - Professor Abdala's GitHub handle is `kurlee`

    2b. You must invite the course assistant(s) to collaborate on your private repository
        \
         - CA Ritik Roongta's GitHub handle is `racro`
         \
         - CA Vignesh Nadar's GitHub handle is `Vignesh-Nadar`
         \
         - CA Xiang Me's GitHub handle is `n132`

Timeliness of your submission will be corroborated with your GitHub commit history. Commits made after the assignment's submission deadline will not be considered for grading.

## Introduction
---
Unfortunately it seems your company never learns. Yet again the company has decided to cut costs and hire Shoddycorp's Cut-Rate Contracting to write another program. But after all, your company insists, their *real* strength is web sites, and this time they were hired to create a high quality web site. As usual they did not live up to that promise, and are not answering calls or emails yet again. Just like last time, the task of cleaning up their mess falls to you.

The project Shoddycorp's Cut-Rate Contracting was hired to create a web site that facilitated the sale, gifting, and use of gift cards. They seemed to have delivered on *most* of the bare funcitonality of the project, but the code is not in good shape. Luckily John Ryan Allen (KG) has read through the code already and left some comments around some of the lines that concern him most. Comments not prefaced by KG were likely left by the original author. Like with all of Shoddycorp's Cut-Rate Contracting deliverables, this is not code you would like to mimic in any way.

## Part 0: Setting up Your Environment
---
Generate the database on which the Django web application project for this assignment will rely, using the following commands:

```
cd ~/AppSec2/GiftCardSite
python3 manage.py makemigrations LegacySite
python3 manage.py migrate
bash import_dbs.sh
```

With your project source files in place and your database configured, you can now run your Django webserver and database at any time, using the following command:
```
sudo python3 manage.py runserver 80
``` 

If you have completed the instructions successfully, you should be able to open a web browser in your virtual machine (FireFox is available in the main menu), and then type in the following web address to visit the website we will be evaluating for this assignment:
```
http://127.0.0.1/
```

You should also see the following output at the bottom your terminal output:
```
Django version 4.0.1, using settings 'GiftcardSite.settings'
Starting development server at http://127.0.0.1:80/
Quit the server with CONTROL-C.
```

Congratulations! The web application is function. But, is it secure?

# Part 1: Auditing and Test Cases
Now that your program is running and you have access to its source code, you can use static and dynamic analysis techniques to review the application for common weaknesses.

There are four (4) intentional weaknesses - among others - embedded into the application. Find one instance of each of the following types of vulnerabilities, and then exploit them as described.

These attacks can take the form of a supplied URL, a POST made to the 
web page, a gift card file, a web page, a javascript function, or some
other method of attack. To create your attacks, you may want to look at 
the HTML source code of the templates and the code of each view, and 
find a way they can be exploited. HTTP proxy tools, such as Burp Suite can help to analyze the raw HTTP requests and understand the attack surface better, but they are not required.

## Task 1 (18pts): Cross-Site Scripting (XSS)
---
Learn about XSS here: https://portswigger.net/web-security/cross-site-scripting

   Find a GET or POST parameter that is vulnerable to reflected cross-site scripting vulnerability. 

**Task 1.a:** Describe the technique(s) used to find and confirm the presence of the vulnerability. As a proof-of-concept exploit, log into the website and submit a JavaScript payload that generates an alert box containing the authenticated user's cookie data. Were you able to obtain the user's cookie with JavaScript? Explain why or why not. Include screenshots of exploitation and snippets of relevant, vulnerable source code.

**Task 1.b:** Using the Python `requests` library, write script that will check for the presence of the vulnerability. The script should send an HTTP request to the web server with a payload that triggers the vulnerability, and then it should parse the web server's response for any indication that the vulnerability was successfully exploited. If the the vulnerability is present, the script should simply print "Vulnerable to XSS!" Save the file as `<NetID>-xss.py` in the root of your repository. Run the script and show its output.
> **Remember!** Your script will first need to perform a login task to create a session if the vulnerability is only exploitable while logged in.

**Task 1.c:** Modify the source code to mitigate the vulnerability identified. Describe the modifications, including specific source code snippets and related filenames affected, and describe why they are effective against the weakness. Do not remove the reflected value; santize the input.

**Task 1.d** Update `<NetID>-xss.py` and modify the output to conditionally print "Not vulnerable to XSS!" if the vulnerability is not successfully exploited.  Run the script and show its output.

## Task 2 (18pts): Cross-Site Request Forgery (CSRF)
---
Learn about CSRF here: https://portswigger.net/web-security/csrf.

Register two (2) distinct user accounts on the web application, one to serve as the target user (<NetID>-target) and one to serve as the attacking user (<NetID>-threat).  Find a feature that allows users to send a gift card to another user, and confirm that the feature can successfully send a gift card between the two accounts (using one regular browser and one InPrivate/Incognito browser - one for each user - is a good technique for logging into the application with two different users).

**Task 2.a:** Describe the technique(s) used to find and confirm the presence of the vulnerability. As a proof-of-concept exploit, create an HTML exploit that abuses cross-site request forgery to coerce a transfer from the target user to the attacking user. Confirm the exploit works by executing the exploit in the target user's browser, and then verifying receipt of the gift card in the attacking user's browser. 

**Task 2.b:** Using the Python `requests` library, write script that will check for the __potential__ presence of the vulnerability. Due to the nature of CSRF, your approach will be slightly different than that for the XSS vulnerability, and you will check for the presence of a mitigating token.  The script should send a routine HTTP request to the affected web resource, and then it should parse the web server's response for the presence of an HTML element containing `csrfmiddlewaretoken` within it. If the the mitigating control is not present, the script should simply print "Vulnerable to CSRF!" Save the file as `<NetID>-csrf.py` in the root of your repository. Run the script and show its output.

**Task 2.c:** Modify the source code to mitigate the vulnerability identified. Describe the modifications, including specific source code snippets and related filenames affected, and describe why they are effective against the weakness. There are two key changes to make - one related to the transmission of cookies and the other related to the token described in Task 2.b.

**Task 2.d** Update `<NetID>-csrf.py` and modify the output to conditionally print "Not vulnerable to CSRF!" if the vulnerability is not successfully exploited.  Run the script and show its output. Explain why the technique employed by the script to determine the state of vulnerability may not be ideal.

## Task 3 (18pts): Structured Query Language Injection (SQLi)
---
Learn about SQLi here: https://portswigger.net/web-security/sql-injection

The application seems to process gift cards in a way that is vulnerable to SQL injection. Buy a gift card, and open the gift card file. One of the JSON parameters' values within it will ultimately be improperly processed by the application when you use the card.

 **Task 3.a:** Describe the technique(s) used to find and confirm the presence of the vulnerability. As a proof-of-concept exploit, construct an SQLi attack string that will be used as malicious input within the gift card file. Confirm the exploit works by attepting using the card within the application and retrieving the password hash for your own user account as well as the `administrator` user account. Include screenshots and ensure to denote the specific field/parameter that was vulnerable, along with the exact attack string used to carry out the attack. Save the malicious gift card file as `<NetID>-sqli.gftcrd` in the root of your repository. 

**Task 3.b:** Using the Python `requests` library, write script that will check for the presence of the vulnerability. The script should send an HTTP request to the web server with a payload that triggers the vulnerability, and then it should parse the web server's response for any indication that the vulnerability was successfully exploited. If the the vulnerability is present, the script should simply print "Vulnerable to SQLi!" Save the file as `<NetID>-sqli.py` in the root of your repository. Run the script and show its output.
> **Remember!** Your script will first need to perform a login task to create a session if the vulnerability is only exploitable while logged in.

**Task 3.c:** Modify the source code to mitigate the vulnerability identified. Describe the modifications, including specific source code snippets and related filenames affected, and describe why they are effective against the weakness.

**Task 3.d** Update `<NetID>-sqli.py` and modify the output to conditionally print "Not vulnerable to SQLi!" if the vulnerability is not successfully exploited.  Run the script and show its output.

## Task 4 (18pts): Command Injection (SQLi)
---
Learn about command injection here: https://portswigger.net/web-security/os-command-injection

The `useCard` resource also appears to be vulnerable to an arbitrary command injection vulnerability whenever a gift card file is used that contains an invalid JSON structure. Review the `extras.py` and `views.py` to determine the vulnerable parameter.

 **Task 4.a:** Describe the technique(s) used to find and confirm the presence of the vulnerability. As a proof-of-concept exploit, capture an HTTP request while using a gift card, and then modify the request to execute arbitrary commands by abusing the vulnerable parameter. Confirm the exploit works by reviewing the command terminal where you launched the `runserver` command earlier. If successful, you should see your command output among the logs (`ls -la` is a good command to inject that will be easily visible).
 
 Use the reverse shell technique in the last section of Lab 1 to inject a command to obtain a reverse shell on the web server. Once established, execute the the following command:
```
hostname; date; id;
``` 
Include screenshots and ensure to denote the specific field/parameter that was vulnerable, along with the exact attack string used to carry out the attack. Also be sure your screenshot of the command above includes the netcat command that was used to set up the listener for the reverse shell. Save the malicious gift card file as `<NetID>-sqli.gftcrd` in the root of your repository.

_Note._ For this proof-of-concept, you are likely obtaining a reverse shell on the same system as the web server is running. Adjust the IP address accordingly. Additionally, you may need to wrap your injected command as an argument to `bash` (a la `bash -c "echo Command Injection!"`) if you see errors similar to `sh: 1: cannot create . . . Directory nonexistent`, indicating execution within a shell that may not support the command.

**Task 4.b:** Using the Python `requests` library, write script that will check for the presence of the vulnerability. The script should send an HTTP request to the web server with a payload that triggers the vulnerability, and then it should parse the web server's response for any indication that the vulnerability was successfully exploited. If the the vulnerability is present, the script should simply print "Vulnerable to CMDi!" Save the file as `<NetID>-cmdi.py` in the root of your repository. Run the script and show its output.
> **Remember!** Your script will first need to perform a login task to create a session if the vulnerability is only exploitable while logged in.

**Task 4.c:** Modify the source code to mitigate the vulnerability identified. Describe the modifications, including specific source code snippets and related filenames affected, and describe why they are effective against the weakness.

**Task 4.d** Update `<NetID>-cmdi.py` and modify the output to conditionally print "Not vulnerable to CMDi!" if the vulnerability is not successfully exploited.  Run the script and show its output.

# Part 2: Automated Regression & Database Encryption
With several vulnerbailities identified, exploited, and mitigated, we can focus on stepping up the security maturity of our application stack - from development automation to database storage.

## Task 5 (10pts): Continuous Integration and Development
---
Now that we have automated the ability to identify weaknesses in the application, we want to make sure there are no regressions in future developments.

Using GitHub Actions, automate the execution of the four automated vulnerability testing scripts you created in tasks 1 through 4.

- `<NetID>-xss.py`
- `<NetID>-csrf.py`
- `<NetID>-sqli.py`
- `<NetID>-cmdi.py`

The resulting GitHub Actions log should show an individual log - with a checkbox - for each test, each of which should be able to be expanded to show the result of the selected test (a la `Vulnerable to ...!` or `Not vulnerable to ...!`)

Ensure your final workflow file is saved to your repository, in `.github/workflows/<NetID>-regression.yml`.

You can learn about GitHub Actions at https://docs.github.com/en/actions/learn-github-actions.

## Task 6 (18pts): Encrypting the Database 
---
The web application's back-end database contains valuable gift card
data. If a threat actor gains unauthorized access to the gift card data, they can use it to obtain free merchandise, or even pay off their
tuition with the NYU tuition gift cards. 

**Task 6.a:** Implement database encryption controls to ensure a compromised gift card entry in the database is not immediately usable without decryption or cracking techniques. Using the `django-cryptography` library, encrypt all of the sensitive fields in the `Cards` table. Once complete, take a screenshot of the `Cards` table, showing the encrypted field values. Finally, demonstrate that the application still works by purchasing and using a gift card after the encryption is in place.

**Task 6.b:** Assume that you have recently discovered that the decryption key for your database encryption has been compromised. Document the process for rotating your encryption key, and then show a screenshot of your `Cards` table again, showing the encrypted field values. Describe technically specific precautions you can take in the future to mitigate unauthorized access to your symmetric key.