# 2024-08-12
## 19:35 - Assessing the issue
- For some reason some of the pages on app.mumie.health can't be updated, whilst others can. Very peculiar, I'm not really sure what's happening here to prevent this!
- I can get on and update the `Welcome to Mumie` page. But not the Constipation (MOT) page.
- I'm going to record the pages I can edit and the pages I can't:

### Can edit:
- Welcome to Mumie (No beaver builder)
- About Us (No beaver builder)
- Baby Blues (No beaver builder)

### Can't Edit:
- Back To Work MOT (had beaver builder running)
- BBtest (had beaver builder running)
- Bowel and Bladder Health  MOT (had beaver builder running)

- Weirdly, I can't edit the beaver builder stuff when I click on the page directly, but I do have access to quick edit and the beaver builder directly from the pages location. This is very peculiar I have no idea why it's happening...?

## 19:47 - The potential isssue
- Okay, so I've had a play around.
- I can edit pages directly in the main editing page that you get when you click the post only if they don't have beaver builder installed on them when I first go in to edit them.
- Any posts I go to update from that editing page that do have beaver builder enabled fail.
- However, if I hover over the beaver builder or quick edit parts of the page, for some reason I can edit them/ get onto the beaver builder and update them. I'm not really sure why this might be.
- Because it's only occurring on posts that have beaver builder enabled on them I'm inclined to think it's an issue with an update to the beaver builder plugin.
- I'll keep looking and see what I can find about it!

## 20:03 - ChatGPT help
- ChatGPT seems to think I should connect directly to the server rather than via cloudfront to see that if that works. So I'll try that and see.
- I've logged onto the server via its IP Address annnnddd, it literally works fine. So that's quite annoying. Clearly something is going on with beaver builder and cloudfront, I'll have to see if I can figure out exactly what.

## 20:22 - Going via `nslookup` to the Load Balancer
- I need to access the load balancer to see if beaver builder and updates work via this location as well and if the issue is well and truly within CloudFront.
- You can find the IP address of a name server with:
```sh
nslookup Mumie-Load-Balancer-1983207497.eu-west-2.elb.amazonaws.com
```
- That's what I used to lookup my load balancer anyway.
- `nslookup` is short for Name Server Lookup... probably.
- Okay, I've one via the load balancer as well now. It definitely works absolutely fine, it's only the CloudFront version that's having issues. So I guess the only thing to do now is switch on logging and exam what's going on.

## 21:04 - Enabling WAF logs
- I've enabled WAF logs for cloudfront so I can figure out exactly what's going wrong when I attempt to do the beaver builder request, but it fails!
- Apparently the logs are kept for 14 days by default initially.

## 22:02 - After enabling
- I've enabled them but you get kind of shit output just from the logs alone within CloudFront, you have to go into CloudWatch and start reading the logs.
- For some reason the logs are on North Virginia only, oh well, I've searched them and found some blocked requests.
- You can go to Log Insights to see what's happening:
![[Pasted image 20240812220629.png]]
- Then you can click on Log Insights:
![[Pasted image 20240812220721.png]]
- And you get to the following screen:
![[Pasted image 20240812220744.png]]
- This is where I've filtered from to find the last 1,000 logs, sort in descending order of time and figure out where I've been blocked!
- Running a query, which you can learn how to do by clicking info:
![[Pasted image 20240812221025.png]]
- helped me find the request I was looking for, I matched the ID that I had from the 403 forbidden CloudFront return that I originally had:
![[Pasted image 20240812221104.png]]
- And I found that basically I was blocked because they thought I was trying to do some Cross Site Scripting, which obviously I'm not.
- I asked my local phi3 about this and it gave me a decent answer about what it thought, but I don't think it's that good, I'm going to get the llama3.1 8b and see what that says instead.

## 22:46 - Talking to Llama3.1 8b
- I spoke to Llama3.1 8b to check what it thinks of the log I'm seeing and why I might be getting blocked.
- It suggested I turn off the beaver builder plugin and have a look again to see if the issue still persists. I suppose I could do that, but it might break parts of my site, I should ask if that could potentially cause some issues.
- Llama3.1 suggested turning off all the plugins to see what happens. I did that it did nothing, I still got the 403 Forbidden error.

## 23:14 - ChatGPT on the likely reason I'm getting blocked
### **Why WAF Blocks This**

- **Prevention of XSS Attacks**: The primary purpose of WAF is to block potential XSS attacks before they reach the server or user’s browser. Since `onerror` combined with inline JavaScript is a known technique for such attacks, WAF blocks requests containing this pattern to prevent a potential security breach.
- **String Matching**: WAF uses pattern matching to detect threats. Even if the use of `onerror` is legitimate, WAF doesn’t have the context to determine this and errs on the side of caution by blocking the request.

- I'm leaving the rest of the conversation to be picked up at a later date, you can find it here: https://chatgpt.com/c/a580c3bb-1df3-4ec0-b341-0547e99413b6
- But also this is the text from the final bit in case you're interested:
```
To set up AWS WAF to allow specific requests (like those containing `onerror` in the body) only when they originate from an admin of your WordPress site, you would need to implement a solution that can reliably identify requests made by admins. This generally involves creating custom WAF rules that allow certain patterns or actions based on conditions that only match for admin users. Here’s how you could approach it:

### 1. **Identify Admin Requests**

- **Authentication Tokens/Cookies**: Admins are typically identified by authentication tokens or cookies (like WordPress’s `wp_logged_in`, `wordpress_sec`, or `wordpress_logged_in` cookies). You can use these to determine if a request is from a logged-in admin user.
- **Custom Headers**: If the site is behind a login page or uses a security plugin, you might also have custom headers or other indicators that can distinguish admin users.

### 2. **Create a Custom AWS WAF Rule**

You’ll create a rule that checks for the presence of specific indicators (like admin cookies) and allows the request if it matches. If the request doesn’t match, the default WAF rules (including those that block potential XSS attacks) will apply.

#### Step-by-Step Guide:

#### **Step 1: Create a Condition to Identify Admin Requests**

- **Use Cookie Matching**: Set up a condition to look for the presence of an admin-specific cookie or header.

**Example**:

- If you’re using the `wordpress_logged_in` cookie to identify admin users, you can create a WAF condition to look for this cookie.
- Go to the WAF console and create a new rule based on the **string match condition**:
    1. **Condition Type**: Choose `String match`.
    2. **Match Location**: Select `Cookie` or `Header`.
    3. **Search String**: Input a string or pattern that uniquely identifies an admin, like part of the `wordpress_logged_in` cookie’s value.

#### **Step 2: Create a Custom Rule to Allow Admin Requests**

- Create a rule that allows requests with the identified admin cookies or headers. This rule should be evaluated **before** the managed XSS protection rule.

**Example**:

- Go to your WAF ACL and add a new rule:
    1. **Rule Type**: Choose `Allow`.
    2. **Conditions**: Add the condition created in Step 1.
    3. **Action**: Set the rule to allow the request if the condition is met.

#### **Step 3: Adjust the Order of Rules**

- Ensure that the custom rule you created to allow admin requests is placed **above** the managed rule that detects and blocks XSS attacks. AWS WAF processes rules in the order they appear, so placing the admin rule first ensures it’s evaluated before the XSS detection rule.

**Rule Order**:

1. **Custom Admin Allow Rule** (based on cookies/headers).
2. **AWS Managed XSS Rule** (e.g., `CrossSiteScripting_Body`).

#### **Step 4: Test and Validate**

- Test the setup by attempting to perform the action that was previously blocked (e.g., updating a page in Beaver Builder) while logged in as an admin. Ensure the request goes through without triggering the WAF block.
- Also, test the same request as a non-admin to verify that the WAF rule correctly blocks it.

### 3. **Additional Considerations**

- **Security Implications**: Ensure that the identification method for admin users is robust and cannot be easily spoofed. Using secure, unforgeable tokens or cookies is crucial.
- **Limited Exposure**: Consider scoping the rule to only specific URLs or paths where the XSS pattern might legitimately occur. This minimizes the risk of accidentally allowing malicious XSS attempts from other parts of the site.

### Conclusion

By creating a custom WAF rule that identifies admin requests via cookies or headers and allows these requests to bypass the specific XSS detection rule, you can ensure that legitimate admin actions are not blocked by WAF while still protecting the rest of your site from potential XSS attacks.
```

# 2024-08-13
## 20:26 - What was wrong
- I read through all of the logs and properly went through everything, turns out it's an XSS issue with beaver builder trying to send a couple of things at the same time which are together tripping a WAF standard rule.
- I'll sort this out at a later date.