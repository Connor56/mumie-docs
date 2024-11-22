# 2024-08-08
## 23:43 - Initial plan
- I think it's possible to build a free paywall system that scales infinitely for free. So this is how I see it working.
	1. The user completes a gravity form that is integrated with Stripe and subscribes them to some payment thing in Stripe.
	2. By signing up, if the user is successful, they'll be added to the Stripe platform and Stripe will email them saying congrats you've signed up for Mumie etc.
	3. Then, every time the user tries to get onto a page there will be a function that checks the last time the user's stripe subscription was checked.
	4. If it was checked less than 24 hours ago it goes with whatever value is held there now.
	5. If it was checked over 24 hours ago it checks the users subscription on stripe again.
	6. If the subscription is active on Stripe then whatever content the user's trying to see will be shown to them.
	7. If the subscription is inactive, they user will be taken to the gravity form page again to sign up for an account via Stripe.
	8. The content that a user is allowed to see will be controlled by PHP rules.
	9. Or the content will be controlled by something like content controller and user roles.
	10. Tbh, I prefer the version with PHP, but that prevents people like Laura and Sarah coming and making changes easily.

- So, what do I have to do to test everything?
	- [x] Set up an account on stripe, I have to do this anyway.
	- [x] Create a local WordPress deployment on my own machine.
	- [x] Setup a test page that only administrators can access on the WordPress site.
	- [x] Check if I can a new table to the database.
	- [x] Check if I can read from the mariadb or mysql (whatever) database from the page in PHP.
	- [x] On that part of the site add some functionality that uses the Stripe API.
	- [ ] Potentially setup a staging site for doing these changes, maybe look into WP Staging or something similar. I would need to know how this worked before doing this for sure.
	- [ ] Probably set up a daily backup thing on AWS first before making changes if one isn't happening already on the instance.
	- [x] Setup a subscription account thing on stripe and put my email down as a member.
	- [x] Check if I can make a conditional function that checks if my email is subscribed in stripe or not.
	- [ ] If I can, check if I can add something to the database for each user that stores this last checked information on them.
	- [ ] See if I can use that in the conditional too.
	- [x] Do some conditional rendering inside of WordPress to see what I can do about showing only part of the text, and a modal sign posting the user to go to the gravity form to sign up via stripe.
	- [x] Check if I even need the gravity form of it might be simpler to just say, go to this stripe screen and do it there, rather than have the form built into the website. It could be a lot cleaner.
	- [x] If all of this is possible, check how I could automatically setup a function that says whether someone is allowed to see content or not based on their premium membership status.

# 2024-08-09
## 09:10 - Testing Stripe
- I'm working on the a test account on Stripe to test the API. I don't know how to put data on there yet though.
- So it turns out there's this tool called composer that manages PHP dependencies: https://getcomposer.org/
- I suppose I should probably get that, it's got 28.5k stars on github, which makes me think it's pretty legit.
- Anyway, if I get that I can install the `stripe/stripe-php` module which I need for working with Stripe directly in Wordpress.
- I suppose I could also create a bespoke plugin too... that's definitely an option.
- I'll look into this.
- This is potentially an article that could help: https://www.dreamhost.com/blog/how-to-create-your-first-wordpress-plugin/
- Alternatively I could code it straight into the theme...? That's always the other option. I guess it's worth checking how good of an option that is.
- I would probably have to start a local development site and test things on there as well. We'll see.

## 17:49 - Local Development
- I've decided that raw dogging such a big change like a paywall on the mumie live site might be a bit too much, so I'm going to rip the site down and deploy locally instead.
- I've been thinking about how to do it best and decide to use local WordPress to do it. I just need to figure out the best way to do that.
- So I'm going to start another saga and test. [[Local Deployment For Development]].

# 2024-08-10
## 13:19 - Setting up the test page and Database
- I need a test page to try out some of the other things I want to get done.
- I'm setting one up in the local site but I need to know if I can edit the database straight from php code on the wordpress site.
- ChatGPT seems to think I should set up a new table within my pre-existing database and hold all of the payment information there.
- Okay, I'm installing vim so that I can save the SQL script that I want to use to produce the table I need for me to track the user paywall stuff myself.
- Yep, I downloaded vim via choco with the following command:
```sh
choco install vim
```
- I did that in an administrator shell, seemed to work perfectly fine!
- Then I opened a new file on a powershell terminal using:
```sh
vim create_user_payments_table.sql
```
- Then when I was in there I pasted in this:
```sql
CREATE TABLE user_payments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    stripe_customer_id VARCHAR(255) NOT NULL,
    stripe_payment_id VARCHAR(255),
    payment_status VARCHAR(50) NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(10) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_checked DATETIME DEFAULT NULL,
    FOREIGN KEY (user_id) REFERENCES wpjh_users(ID)
);
```
- This should be able to produce the database table for me that stores information on user payment history and current status etc.

# 2024-08-14
## 00:33 - Loading create table onto the site
- You can load a file onto your mysql with the following command:
```sh
mysql -u username -p database_name < your_file.sql
```
- Fortunately on the local version of the site there's no password required. So just this will do:
```sh
mysql < create_user_payments_table.sql
```
- I got an error saying no database was selected when I tried to run this. So what's the solution?
- Also, I tried to do it on my PowerShell version, but the piping operator `<` doesn't work on there. Watch out for that.
- Okay I get the following error when I try to pipe in the .sql file:
```sh
ERROR 1824 (HY000) at line 3: Failed to open the referenced table 'wpjh_users'
```
- Apparently, this is because I'm using the MyISAM engine type for the `wpjh_users` table.
- This is fine, but means I can't use the foreign key stuff without switching to InnoDB. Doing this feels like a big deal so I'm not going to, I'm just going to change to not use a foreign key and see what happens.
- The new code is this:
```sql
USE local;

CREATE TABLE user_payments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    stripe_customer_id VARCHAR(255) NOT NULL,
    stripe_payment_id VARCHAR(255),
    payment_status VARCHAR(50) NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(10) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_checked DATETIME DEFAULT NULL
);
```
- This ran without error.
- Now I can get onto mysql and check the database is there with:
```sh
> mysql
>>> use local
>>> show tables;
```
- And she's there! Bingo baby. Now to see about adding data to it.
- I shut it down and started it back up again, the table persisted.

## 09:35 - Testing VSCode locally
- I just tested to see if I make a change to a page whether it immediately gets updated on the live locally deployed site.
- It does! That's awesome, that means I should be able to test everything quickly and easily as I go through and change things.
- First order of business is to create a new button on this site that says sign up via stripe and takes people out to a stripe page where they can pay zero pounds to get a subscription.

## 23:16 - Getting a query running
- ChatGPT has given me some code to run that should let me make a connection to a database in SQL:
```php
<?php
	$servername = "localhost";
	$username = "";
	$password = "";
	$dbname = "";

	// Create connection
	$conn = new mysqli($servername, $username, $password, $dbname);

	// Check connection
	if ($conn->connect_error) {
		die("Connection failed: " . $conn->connect_error);
	}
	?>
```
- The issue initially was that I didn't know what the credentials etc. should be, but now I've asked ChatGPT again it has rightly told me to go and look at the `wp-config.php` file and see!
- Christ, what would we do without you ChatGPT.
- Anyway, so this is the new code that works:
```php
<?php
	$servername = "localhost:10011";
	$username = "root";
	$password = "root";
	$dbname = "local";

	// Create connection
	$conn = new mysqli($servername, $username, $password, $dbname);

	// Check connection
	if ($conn->connect_error) {
		die("Connection failed: " . $conn->connect_error);
	}
	?>
```
- Now I just need to figure out how to read everything from that connection.
- Okay, I now have some extra code that can create a record and some more code that can read all the records from the `user_payments` table!
- Here we go:
```php
// SQL to insert a new record into the user_payments table
	$sql = "INSERT INTO user_payments (user_id, stripe_customer_id, stripe_payment_id, payment_status, amount, currency, created_at) 
VALUES (1, 'cus_123456789', 'pi_987654321', 'completed', 99.99, 'USD', NOW())";

	if ($conn->query($sql) === TRUE) {
		echo "New record created successfully";
	} else {
		echo "Error: " . $sql . "<br>" . $conn->error;
	}

	$sql = "SELECT * FROM user_payments";
	$result = $conn->query($sql);

	if ($result->num_rows > 0) {
		// Output data of each row
		while ($row = $result->fetch_assoc()) {
			echo "id: " . $row["id"] . " - User ID: " . $row["user_id"] . " - Stripe Customer ID: " . $row["stripe_customer_id"] .
				" - Payment Status: " . $row["payment_status"] . " - Amount: " . $row["amount"] . " " . $row["currency"] .
				" - Created At: " . $row["created_at"] . "<br>";
		}
	} else {
		echo "0 results";
	}
```
- This is awesome because it basically means I'm halfway there to making this work. Now I just need the ability to query stripe and to understand how the stripe API works.
- I can do this in bed as I go to sleep, read about what's possible in the API.
- Awesome stuff though, I think this is a path to things working, I have to record something in the stripe API somehow, and identifier for the user so that I can use it to check they are the person they say they are.
- Here's what all the code together on my `test_admin_page` looks like:
![[Pasted image 20240814232638.png]]
- So funny, every time I reload a new entry gets put in. But it's just so awesome to see PHP do this, really cool software and it makes me think PHP is the boss!

# 2024-08-15
## 20:23 - Stripe stuff
- I have confirmed I can access the database in PHP and do stuff there. Geddon son.
- But next I have to figure out how to use the Stripe API, which is non-trivial because they're documentation is shit. But in line with what I just posted on medium, I may was well just smash it out and see what I can get going. So here goes!

## 21:05 - Installing composer
- Like normal all these tools have their preferred way for you to install them, the way PHP composer wants you to do it is pretty annoying.
```sh
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```
- They want me to do all this shit...? What the fuck!
- Why do I have to do that? Any time someone asks me to bareback something like that into my terminal I'm immediately suspicious.
- No thanks. I'll try `choco install` first!
```sh
choco install composer
```
- much nicer.
- So I've installed composer and it also installed a version of PHP, but I guess not the version of PHP that my local site is running... hmmm not ideal! 
- Maybe I will have to do the gay php composer install! It's always the way.
- Yep, looks like it.
- The thing is, it's all fine I guess, but it's just odd when you're asked to whack something into the terminal that you don't understand. As it happens, the `-r` just means `run` this code in the quotations.
- They're saying I should move it somewhere on the path but they're just getting fancy now, I cba to do that, I'll use it from here!
- I could look into this, but it would require me looking into how to add things to your path in windows quickly, which I can't really be fucked to do right now.

## 21:25 - Downloading the Stripe API
- So I've got composer now I just need to get the Stripe API. According to [their GitHub](https://github.com/stripe/stripe-php), all you have to do is run the following command:
```sh
composer require stripe/stripe-php
```
- It worked! First time! Not that common of an occurrence if we're being honest! So here we go:
```sh
./composer.json has been created
Running composer update stripe/stripe-php
Loading composer repositories with package information
Updating dependencies
Lock file operations: 1 install, 0 updates, 0 removals
  - Locking stripe/stripe-php (v15.6.0)
Writing lock file
Installing dependencies from lock file (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
  - Downloading stripe/stripe-php (v15.6.0)
  - Installing stripe/stripe-php (v15.6.0): Extracting archive
Generating autoload files
No security vulnerability advisories found
Using version ^15.6 for stripe/stripe-php
```
- That was the output!
- Well, that's that bit done, took about 2 minutes. Beautiful. Onto the next!
- I've updated jira with a bunch of the tasks I need to complete, you can see it here: https://mumie-test.atlassian.net/jira/software/projects/MWD/boards/1/backlog?epics=visible&issueParent=10087&selectedIssue=MWD-90

## 21:34 - Pinging the Stripe API
- I've got the following code from ChatGPT, let's see if it works:
```php
<?php

	require 'vendor/autoload.php'; // This loads the Stripe PHP library
\Stripe\Stripe::setApiKey('your_secret_key'); // Replace with your actual secret key

	// Retrieve the customer using their ID
	$customerId = 'cus_fakecustomerid'; // Replace with your actual customer ID

	try {
		$customer = \Stripe\Customer::retrieve($customerId);
		echo 'Customer ID: ' . $customer->id . '<br>';
		echo 'Email: ' . $customer->email . '<br>';
		echo 'Description: ' . $customer->description . '<br>';
	} catch (\Stripe\Exception\ApiErrorException $e) {
		// Handle error appropriately
		echo 'Error retrieving customer: ' . $e->getMessage();
	}

	?>
```
- I'm going to try the publishable token first for my API:
![[Pasted image 20240815214316.png]]
- Look how f'ing easy that was!
![[Pasted image 20240815214351.png]]
- That says:
```php
Error retrieving customer: This API call cannot be made with a publishable API key. Please use a secret API key. You can find a list of your API keys at https://dashboard.stripe.com/account/apikeys
```
- I mean, come on! So close already.
- Yep, with the secret key added this thing almost works, just need to give a real customer ID.
- So that's that sorted, now I just need to see about creating a context window or whatever you call it that springs up and holds customer meta data from a call to the database asking for my personal details or what have you.

## 22:51 - Setting up a Stripe payment window thing
- Okay, I just did a lot of LLM'ing and Googling and what have you to learn a bit more about PHP and it's ecosystem.
- A llloooott of quizlet cards made. Anyway, the long and short of is I got to setting up the Stripe window, and the code to do it is simply this:
```php
$session = \Stripe\Checkout\Session::create([
		'payment_method_types' => ['card'],
		'line_items' => [[
		  'price_data' => [
			'currency' => 'usd',
			'product_data' => [
			  'name' => 'T-shirt',
			],
			'unit_amount' => 2000,
		  ],
		  'quantity' => 1,
		]],
		'mode' => 'payment',
		'metadata' => [
		  'order_id' => '6735',
		  'user_id' => '1234',
		  'custom_note' => 'This is a custom field',
		],
		'success_url' => 'https://yourdomain.com/success',
		'cancel_url' => 'https://yourdomain.com/cancel',
	  ]);
```
- How about that, matched with my API Key this basically takes you there to the page that gets you to setup your details for payment! Cool huh!
![[Pasted image 20240815230103.png]]
- Pretty goddamn easy if I'm honest! Now I just need to figure out how to make this work and look really nice.

## 23:30 - Yeah, not so easy
- I've been trying to see how to do this for a little while now. Not too much help, but I did find this API setup thing: https://docs.stripe.com/api/subscriptions/create?lang=php
- Really good resource actually probably worth perusing in the future!
- Here's the API reference for creating a Session: https://docs.stripe.com/api/checkout/sessions/create?lang=php
- I'm using this code currently to setup a recurring subscription page for Mumie monthly:
```php
$session = \Stripe\Checkout\Session::create([
		'payment_method_types' => ['card'],
		// 'customer_email' => $current_user->user_email,
		'line_items' => [[
		  'price_data' => [
			'currency' => 'gbp',
			'product_data' => [
			  'name' => 'Mumie Monthly Subscription',
			],
			'unit_amount' => 499,
			'recurring' => [
				"interval" => "month",
				"interval_count" => 1,
			],
		  ],
		  'quantity' => 1,
		]],
		'mode' => 'subscription',
		// 'metadata' => [
		//   'order_id' => '6735',
		//   'user_id' => '1234',
		//   'custom_note' => 'This is a custom field',
		// ],
		'success_url' => 'https://app.mumie.health',
		'cancel_url' => 'https://app.mumie.health/your-gp-postnatal-check',
	  ]);
	  
	  header('Location: ' . $session->url);
```
- This works and gets me basically what I want. I think I just need something like a page before hand that lets the user select their subscription type on a Mumie like page and then get taken to Stripe to finish the subscription and go back home.
- Apparently you can use the following to search the Stripe customers you have:
```php
$stripe = new \Stripe\StripeClient('your_secret_key');

$result = $stripe->customers->search([
  'query' => 'name:\'Jane Doe\' AND metadata[\'foo\']:\'bar\'',
]);
```
- Pretty cool eh! This works according to ChatGPT too, and I took it from the Stripe API page: https://docs.stripe.com/api/customers/search?lang=php
- It comes with its own neat little query language!

## 00:12 - Working late on search
- I'm trying to get the customer search functionality to work, but it doesn't really seem to want to be working. I'm going to try negating one of the terms, which apparently you can do by putting a `-` in front of it, see if I get anything at all.
- I see, the issue was that I had completed the payment as a guest not a customer! That was why it was failing. I'll try again as a guest and see.
- This is a place you can search subscriptions: https://docs.stripe.com/api/subscriptions/search?lang=php
- Okay, cool, I think that's enough for one day. Almost getting there on this API and payment stuff! Should have something set up very very soon :D

# 2024-08-16
## 20:00 - The tasks to complete to finish the paywall
- I set up a bunch of these tasks in the Mumie slack last night, here they are:
	- [x] Figure out how to make the checkout session record someone as a customer.
	- [x] Figure out where subscriptions go on the stripe api.
	- [x] Figure out how to search subscriptions and their associated customer.
	- [x] Figure out if customer data is important if we already have subscription data.
	- [x] Figure out how to pass information from a subscription membership choice page to the Stripe API.
	- [x] Figure out how to let user's cancel subscriptions if they wish.
	- [x] Build two dummy pages: first has subscription options, second redirects to stripe checkout with user data.
	- [x] If second page is accessed via any means but the first page, redirect to last page seen.
	- [x] Test searching for customer or subscription via metadata.
	- [x] Test setting up hash key for user that can be linked to a subscription or customer and therefore lets you know if they have an active subscription or not.
	- [ ] Test having a column in the user_payments table that says how many seconds to wait before checking if a subscription exists again.
	- [ ] Test recording in a column if the subscription is active.
	- [ ] Test resetting that second to wait to 0 when a user has gone to the checkout page. This will add latency, but improve the speed a subscription appears to activate.
	- [ ] Test how long it takes the stripe api to correctly update and show information.
	- [ ] Make sure that when the user comes onto the subscription page the referring page is stored somewhere permanent and not over written by going back and forth to stripe!

- And here's an image of a potential user flow:
![[Pasted image 20240816200159.png]]

## 21:35 - Let's start building
- Subscription page with three buttons first I think.
- I have a shit subscription page! It's amazing what you can build with ChatGPT in a short amount of time!

## 22:19 - Piddling around with CSS 
- The part I'm certainly the worst at... the styling! Lol.
- Well here we go then, I have a shitload of crappy CSS styles left over from the previous dummies who made this heap of shit, and now I have to try and add to the mess with my own styles.
- I'm going to start by putting all my styles into css files that match the name of the php page I'm styling, then I'll enqueue those files in `functions.php` instead, all in one go, rather than fuck around adding to styles.css directly. This also keeps me a bit more modular and a bit less likely to do serious harm.
- This makes me feel like I'm doing component styling in React or something, and overall this just feels a lot nicer.
- I've added the following code to make sure all files ending in `.css` in the template directory are added to the WordPress them:
```php
function enqueue_all_css_files()
{
	// Define the directory containing the CSS files
	$css_directory = get_template_directory() . '/css/'; // Adjust this path to your directory
	$css_url = get_template_directory_uri() . '/css/'; // URL path to the directory

	// Scan the directory for .css files
	$css_files = glob($css_directory . '*.css');

	// Enqueue each found CSS file
	$bad_styles = ["style-layout2.css", "links.css"];
	foreach ($css_files as $css_file) {
		if (!in_array($css_file, $bad_styles)) {
			continue;
		}
		$file_name = basename($css_file, '.css'); // Get the file name without extension
		$handle = 'theme-' . $file_name; // Create a unique handle for each stylesheet

		wp_enqueue_style($handle, $css_url . basename($css_file), array(), null, 'all');
	}
}
```
- This seems to work, in that at least it hasn't upset the overall appearance of the site so far.
- I have, of course, had to add a list of `$bad_styles` these are styles that I guess aren't really supposed to be generally used, or used at all and thus ruin the site when they're involved in styling it in general.

## 22:45 - Figuring out the issues
- Again, there were issues with doing this! Really annoying little issues with enqueueing things and getting used to working with WordPress / PHP, but I'm getting there.
- The code I have above is wrong actually the correct code is:
```php
function enqueue_all_css_files()
{
	// Define the directory containing the CSS files
	$css_directory = get_template_directory() . '/'; // Adjust this path to your directory
	$css_url = get_template_directory_uri() . '/'; // URL path to the directory

	// Scan the directory for .css files
	$css_files = glob($css_directory . '*.css');

	// Enqueue each found CSS file
	$bad_styles = ["style-layout2.css", "links.css"];
	foreach ($css_files as $css_file) {
		if (in_array($css_file, $bad_styles)) {
			continue;
		}
		$file_name = basename($css_file, '.css'); // Get the file name without extension
		$handle = 'theme-' . $file_name; // Create a unique handle for each stylesheet

		wp_enqueue_style($handle, $css_url . basename($css_file), array(), null, 'all');
	}
}

// You have to add this to get WordPress to add the CSS to all sites or something.
add_action('wp_enqueue_scripts', 'enqueue_all_css_files');
```
- Once I'd corrected that and added the following CSS from ChatGPT:
```css
.subscription-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
  background-color: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.subscription-container h2 {
  margin-bottom: 20px;
  color: #333;
}

.subscription-option {
  margin-bottom: 20px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #fff;
}

.subscription-option h3 {
  margin-bottom: 10px;
  color: #007bff;
}

.subscription-option p {
  margin-bottom: 15px;
  color: #666;
}

.subscription-option button {
  padding: 10px 20px;
  color: #fff;
  background-color: #007bff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.subscription-option button:hover {
  background-color: #0056b3;
}

```
- I got the following:
![[Pasted image 20240816224719.png]]
- How cool is that! Way nicer than most of the rest of the stuff on the site really! But let's see if we can make it even better.
- Okay, so I've basically got everything I want bar the colour scheme being right, so that's nice!
- I think your man has dun gud:
![[Pasted image 20240816231628.png]]
- Very sexy, very nice 👌👌👌👌
- The CSS that did this is:
```css
/* General Container Styling */
.subscription-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

/* Grid Layout for Subscription Options */
.subscription-container form {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

/* Style Each Subscription Option */
.subscription-option {
  padding: 20px;
  padding-top: 40px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: #f9f9f9;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: transform 0.3s ease, background-image 0.3s ease;
  position: relative;
  overflow: hidden;
}

/* Hover and Active State for Interactivity */
.subscription-option:hover,
.subscription-option:active {
  transform: scale(1.05); /* Slightly increase size on hover */
  background-image: linear-gradient(to right, #6a11cb 0%, #2575fc 100%);
  /* background-image: linear-gradient(to right, #fcb900 0%, #179b91 100%); */
  color: #fff; /* Change text color to white for better contrast */
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15); /* Enhance the shadow on hover */
}

/* Making the Entire Option a Clickable Button */
.subscription-option button {
  display: none; /* Hide the original button */
}

/* Add a Highlight Effect */
.subscription-option::before {
  content: "";
  position: absolute;
  top: 0;
  left: -100%;
  width: 200%;
  height: 100%;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.3),
    transparent
  );
  transition: left 0.5s ease;
}

.subscription-option:hover::before,
.subscription-option:active::before {
  left: 100%;
}

@media (max-width: 768px) {
  .subscription-container form {
    grid-template-columns: 1fr;
  }
}
```

## 23:16 - Working on the backend part now to communicate with Stripe.
- So basically, the way this all works now is there's a piece of JS code that attaches `<input>` tags to a `<form>` element when one of the options is clicked. The inputs are sent as part of a POST request to the checkout page which can then process them to create the Stripe checkout page.
- The JS code that does this is:
```js
document.querySelectorAll('.subscription-option').forEach(option => {
	option.addEventListener('click', function() {
		const form = option.closest('form');
		// Price section
		const input = document.createElement('input');
		input.type = 'hidden';
		input.name = 'price';
		input.value = this.querySelector('button').value;

		// Interval section
		const input2 = document.createElement('input');
		input2.type = 'hidden';
		input2.name = 'sub_interval';
		input2.value = this.querySelector('button').id

		form.appendChild(input);
		form.appendChild(input2)

		form.submit();
		console.log("hi");
	});
});
```
- It's in-lined into the PHP as a `<script>` tag.
- Basically, this is this page largely done, there's still potentially a bit more automation to do, but by and large I'm happy with this page, including the CSS styling.
- Okay, I've got a lot of the backend sorted out as well now with logic telling it to setup the checkout based on the information that it's been sent by the user, and then going back to the subscribe page if they click cancel.
- This looks really nice actually! Really cool that it can work like that xD.
- I've noticed a problem with using the referrer thing, I could go from stripe back to subscription and all of a sudden stripe checkout is the referrer... doh! That wouldn't be too good! So need to make sure it's actually the last page before subscription was clicked on that comes from Mumie!
- I've now basically massively simplified the checkout PHP page too, and it's much simpler, only containing the data I need. Next steps will be putting together the function that checks the database to see if the user has an active subscription or not.
- This is where I'm at for the checkout, for now:
```php
<?php
/*
	Template Name: Checkout
	*/
?>
<?php Starkers_Utilities::get_template_parts(array('html-header', 'header')); ?>

<?php if (is_user_logged_in()) { ?>

	<?php

	// Bounce a user away from the checkout if they haven't come through the
	// subscription page.
	$referrer = $_SERVER['HTTP_REFERER'];
	if (sizeof($_POST) < 2) {
		if ($referrer == "") {
			header("Location: https://app.mumie.health");
		} else {
			header("Location: " . $referrer);
		}
	}

	$price = (int)$_POST["price"];
	$interval = $_POST["sub_interval"];
	$referrer = $_POST["referrer"];

	if ($interval == "1_month") {
		$interval = "month";
		$interval_count = 1;
		$product_name = "Mumie App";
	} else if ($interval == "3_month") {
		$interval = "month";
		$interval_count = 3;
		$product_name = "Mumie App";
	} else if ($interval == "1_year") {
		$interval = "year";
		$interval_count = 1;
		$product_name = "Mumie App";
	} else {
		// TODO: Replace this with a redirect somewhere
		die("No genuine interval selected.");
	};

	if ($referrer == "") {
		$referrer = "https://app.mumie.health";
	}

	require 'vendor/autoload.php'; // This loads the Stripe PHP library

	$current_user = wp_get_current_user();
	$user_hash =  hash("sha256", "app.mumie.health" . $current_user->user_id);

	// Set up the checkout session and you're done!
	\Stripe\Stripe::setApiKey('sk_test_51Plny4LEaxMgsg6xcNHflJfgtGKvHqJF71jG5zgeNu8LocifSwi5QteYfq9luKidiPYekhoNWn6zjZMRlSQC1uDm00f0DXnlcT'); // Replace with your actual secret key
	$session = \Stripe\Checkout\Session::create([
		'payment_method_types' => ['card'],
		'line_items' => [[
			'price_data' => [
				'currency' => 'gbp',
				'product_data' => [
					'name' => $product_name,
				],
				'unit_amount' => $price,
				'recurring' => [
					"interval" => $interval,
					"interval_count" => $interval_count,
				],
			],
			'quantity' => 1,
		]],
		'mode' => 'subscription',
		'metadata' => [
			'user_hash' => $user_hash,
		],
		'success_url' => $referrer,
		'cancel_url' => 'https://app.mumie.health/subscribe',
	]);

	header('Location: ' . $session->url);

	?>

<?php } ?>

<?php Starkers_Utilities::get_template_parts(array('footer', 'html-footer')); ?>
```
- This is a very simple little page that basically sets up everything and when complete won't even bother loading the header or the footer, it will either work or it will redirect to an older page or something, there will be no in-between.

## 00:31 - Still lots to do but getting there
- I think the flow of getting the users into the system is nearly done on a functional level. It needs tidying up of course, and reformatting to look beautiful, but we're getting there.
- You can go from a subscription page to the checkout page and be bounced into a Stripe session that correctly gets you to sign up for the right subscription.
- Next I need to test where that subscription information goes, where the metadata gets stored, and then setup the function that limits a page for a user depending on whether they have an active subscription or not.
- That will be the greatest part of this coming soon!

# 2024-08-17
## 12:25 - The subscribe input flow
- I've got the subscribe page to checkout section down now. All I need to do is test that the metadata is saved correctly and that it can be accessed via a stripe search function.
- I need to put a lot of these tasks onto Jira I think, I should probably do that when I'm back from the outing with Jessie and we've had some food etc.
- The page I need to change apparently is the `single.php` page. When I get back I'll set up what I want that to look like with the content block and paywall, and then test getting through it by creating an account!

## 18:38 - Making `single.php` work
- I need to find a place to put a function that calls out to stripe and figures out if you have permission to view the page.

## 22:40 - Adding to the `functions.php` file
- Turns out, maybe as expected, the place to add a function to for general use is the `functions.php` file.

## 22:31 - Do you really need the prefix?
- What's the point of the table prefix in wordpress? Yes, apparently you do.

## 00:02 - Metadata not being recorded
- For some reason the metadata I'm adding in the checkout isn't being recorded when the subscription is, I need to look into this.
- I need to figure out how to add actual customer metadata to this system because it doesn't seem to want to let you tbh.

# 2024-08-18
## 12:17 - Checking if setting `customer` works
- It looks like metadata isn't stored anywhere functionally useable so I'm going to try and push data into the customer somehow or another with all the options available to me at checkout!
- So I've found the metadata, it can be found in the checkout session, it's attached to the checkout session, pretty annoying.
![[Pasted image 20240818122339.png]]
- I wonder if there's a way around this?
- Okay, sending a string as a customer doesn't work, results in an error. Annoying.
### It Doesn't

## 12:32 - How else can I connect a customer to my internal system then...?
- I guess one option is to create the customer ahead of time and then use the customer key that stripe creates to build the checkout?
- Apparently webhooks are the way forward, I'm actually a big fan of this now I understand them!

## 22:47 - Figure out WordPress webhooks before bed time?
- Apparently there's a PHP file called admin-post.php that you can use for webhooks that something like Stripe can call into.
- I'll see if I can do that by the end of the night.
- This is what Stripe recommends for the webhook code:
![[Pasted image 20240818231458.png]]
- This doesn't seem to work, I've got the code set up to read the incoming request and set it up as WordPress apparently expects me to (according to ChatGPT), but the request isn't being dealt with properly. It seems to do a redirect, which I also see when I try and load the page as a non-user.
- Either the theme is failing some how, or perhaps ChatGPT is wrong, this does seem to be a fairly niche topic.
- I think it could be the site bouncing any traffic that's not signed in to the homepage or something like that, which seems pretty stupid. I don't get it, why would it do that if I can clearly see in `admin-post.php` there are no redirects!
- ChatGPT recommended putting my Webhook into the functions.php file, which I've done, but it makes no difference! `do_action` doesn't seem to be working for me.

## 23:45 - I didn't manage to figure out how this Webhook works.
- So annoying not being able to figure this out. It definitely feels like a much unexplored area of WordPress, I guess I may be forced to ask a question on Reddit in the end.
- So I guess the issue is that before the webhook can get to the PHP code in admin-post it's being redirected to the login or something. So I just need to figure out why/ what is loaded by the site when someone isn't logged in and accesses the wp-admin route.

# 2024-08-21
## 23:21 - I figured out what was probably wrong
- The webhook's not working because it's going to the DNS of the real site, not to my locally hosted site! Duh!
- Any way, to get around this, I can send a request using python via my own machine.
- Okay, so when I run the request locally I'm still getting the re-direct for some reason. However, initially it was failing because of a self-signed certificate, so at least I know it was actually going to the correct site.
- Weird then, that it's still being redirected when it should be giving a 400 or something instead.
- I've set up the request so that it should go directly to admin-post.php and then return a 400 error immediately, but it doesn't.
- I tried switching off all the plugins to see if it made a difference, it didn't make any difference whatsoever.
- The site url and home url are both hardcoded in so they can't be the issue.
- Perhaps it's an issue with the stripe library needing to be loaded or something. Perhaps the action can't be loaded without that.
- I've commented out everything so that nothing from Stripe is required, now we'll see if it works.
- Nope, still a redirection. How do I track why that's happening?
- Apparently it could be from things like `wp_redirect`, so I've commented out the one occurrence of that in the `functions.php` file. It could also be `wp_safe_redirect` or `header` too, but they're not in functions.php.
- Okay, that was the issue in the end then! It was the redirect thing that was occurring. I'll need to look a bit further into that and why that happens as a default.
- This is the code in question in `functions.php`:
```php
add_action('init', 'blockusers_init');
function blockusers_init()
{
 	if (is_admin() && ! current_user_can('administrator') && ! (defined('DOING_AJAX') && DOING_AJAX)) {
 		wp_redirect(home_url());
 		exit;
 	}
}
```
- Apparently it works like this, you want to check if the user is an admin and if they're requesting the admin area. Then if they're not an administrator you deny them access.

## 00:03 - Fixed the issue
- I've changed the redirect to allow through stripe webhook calls to the admin-post.php location, and only if they have an action parameter set correctly for the stripe-webhook:
```php
add_action('init', 'blockusers_init');
function blockusers_init()
{
	// Allow non-admin users to access specific actions, such as Stripe webhooks
	if (is_admin() && !current_user_can('administrator') && !is_stripe_webhook() && !(defined('DOING_AJAX') && DOING_AJAX)) {
		wp_redirect(home_url());
		exit;
	}
}

// Function to detect if the request is a Stripe webhook
function is_stripe_webhook()
{
	// Check if the action parameter is set and matches the one used for Stripe webhooks
	// Check if the script being executed is admin-post.php
	if (basename($_SERVER['SCRIPT_FILENAME']) === 'admin-post.php') {
		// Check if the action parameter is set and matches the one used for Stripe webhooks
		if (!empty($_REQUEST['action']) && $_REQUEST['action'] === 'stripe_webhook') {
			return true;
		}
	}

	// Additional logic can be added here if you need to match other conditions
	// related to Stripe webhooks or other similar requests.

	return false;
}
```
- **chef's kiss** beautiful! Love it, I'm basically there on it.

# 2024-08-25
## 08:53 - Building the paywall
- Okay, I guess this means real testing can begin now.
- I've got a test payload from the webhook sent by Stripe, I'm running this via a request in a jupyter notebook that I've got set up in vscode. It's an easy life!
![[Pasted image 20240825090856.png]]
- The webhook request is huge, you can see it here: https://dashboard.stripe.com/test/webhooks/we_1PpHM9LEaxMgsg6xw1u4LVif
- This is the text of it straight from Stripe:
```json
{
  "id": "evt_1PrbOXLEaxMgsg6x7M8qcI4B",
  "object": "event",
  "api_version": "2024-06-20",
  "created": 1724572913,
  "data": {
    "object": {
      "id": "cs_test_a1WPcLCvin7t1xXX30l9UJ45QDn7eXuZlv3lTmfkrQ4U7IaOVZuFEbhaC9",
      "object": "checkout.session",
      "after_expiration": null,
      "allow_promotion_codes": null,
      "amount_subtotal": 999,
      "amount_total": 999,
      "automatic_tax": {
        "enabled": false,
        "liability": null,
        "status": null
      },
      "billing_address_collection": null,
      "cancel_url": "https://app.mumie.health/subscribe",
      "client_reference_id": "cc0d1c825a224e9ab90cc3fa9059337d7976582bb42f8333dbd00094d44c162e",
      "client_secret": null,
      "consent": null,
      "consent_collection": null,
      "created": 1724572878,
      "currency": "gbp",
      "currency_conversion": null,
      "custom_fields": [
      ],
      "custom_text": {
        "after_submit": null,
        "shipping_address": null,
        "submit": null,
        "terms_of_service_acceptance": null
      },
      "customer": "cus_Qj3VaTMFgdvgsZ",
      "customer_creation": "always",
      "customer_details": {
        "address": {
          "city": null,
          "country": "GB",
          "line1": null,
          "line2": null,
          "postal_code": "EX4 6LY",
          "state": null
        },
        "email": "connor.jira@proton.me",
        "name": "a",
        "phone": null,
        "tax_exempt": "none",
        "tax_ids": [
        ]
      },
      "customer_email": null,
      "expires_at": 1724659278,
      "invoice": "in_1PrbOULEaxMgsg6xQXuekg6Q",
      "invoice_creation": null,
      "livemode": false,
      "locale": null,
      "metadata": {
        "user_hash": "cc0d1c825a224e9ab90cc3fa9059337d7976582bb42f8333dbd00094d44c162e"
      },
      "mode": "subscription",
      "payment_intent": null,
      "payment_link": null,
      "payment_method_collection": "always",
      "payment_method_configuration_details": null,
      "payment_method_options": {
        "card": {
          "request_three_d_secure": "automatic"
        }
      },
      "payment_method_types": [
        "card"
      ],
      "payment_status": "paid",
      "phone_number_collection": {
        "enabled": false
      },
      "recovered_from": null,
      "saved_payment_method_options": {
        "allow_redisplay_filters": [
          "always"
        ],
        "payment_method_remove": null,
        "payment_method_save": null
      },
      "setup_intent": null,
      "shipping_address_collection": null,
      "shipping_cost": null,
      "shipping_details": null,
      "shipping_options": [
      ],
      "status": "complete",
      "submit_type": null,
      "subscription": "sub_1PrbOULEaxMgsg6x0Hiqmtm4",
      "success_url": "https://app.mumie.health",
      "total_details": {
        "amount_discount": 0,
        "amount_shipping": 0,
        "amount_tax": 0
      },
      "ui_mode": "hosted",
      "url": null
    }
  },
  "livemode": false,
  "pending_webhooks": 1,
  "request": {
    "id": null,
    "idempotency_key": null
  },
  "type": "checkout.session.completed"
}
```

## 09:18 - Webhook can hook
- The webhook is correctly connecting via `admin-post.php` into the `functions.php` function that I've set up for handling this.
- Still got this strange issue with not allowing stripe to be loaded.
- I'm going to open up what's allowed through into the stripe webhook to see if I can get through to the error.
- Yeah the error is the same as it was before, wp-admin can't load the vendor packages for some reason, really weird.
- ChatGPT recommended I do this:
```php
require_once(get_template_directory() . '/vendor/autoload.php');
```
- This failed, probably because it's not pointing to the right location now, and also it fails on all other pages too because it's now not pointing to the right location for anything.
- I used the Absolute path to the main directory on my WordPress site and it worked, this is the code required:
```php
require_once(ABSPATH . 'vendor/autoload.php'); // This loads the Stripe PHP library
```
- I figured out where it needed to go by `echo`'ing out the strings I was using to find the vendor packages.
- Okay, everything tidied up, no messy `echo` statements left, time to handle the request properly.

## 09:58 - Webhook comes through
- The payload has come through to mumie now. So all that's left to do is update the database!

## 20:40 - Creating a simpler user payment table
- I'm going to delete the old `user_payments` table and add a simpler one for webhooks. This is what the new code for the table looks like:
```sql
use local;

CREATE TABLE user_payments (
	id INT AUTO_INCREMENT PRIMARY KEY,
	user_id INT,
	user_hash VARCHAR(255) NOT NULL,
	is_subscribed BOOLEAN NOT NULL,
);
```
- For some reason this is causing an issue when I run:
```sh
mysql < create_user_payments_table.sql
```
- I've literally no idea why, so chatgpt it is. This is the error:
```sh
ERROR 1064 (42000) at line 3: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ')' at line 6
```
- Ahhhh, the issue is the trailing comma! They're not allowed, talk about shit error messages though, surely not a hard one to say.
- Okay, there we go! A new table added to do the user payment tracking with.

## 20:54 - Building the Webhook system
- Now I just need to insert the data into the table. This is proving slightly troublesome because I can't figure out what the type of the `$event->type` attribute is.
- My code isn't adding anything to the database unfortunately. Here's the code:
```php
$session = $event->data->object; // Contains the checkout session object
// Handle successful payment here
global $wpdb;
$data = array(
	'user_hash' => $session->client_reference_id,
	'is_subscribed' => true,
);
$format = array('%s', '%b');
if ($session->metadata->type == "membership") {
	echo "hi";
	$wpdb->insert("user_payments", $data, $format);
}
break;
```
- One of the main issues seems to be the table name simply doesn't matter, it doesn't change anything or fail or what have you if I choose a different one.
- Okay, no error message, but I figured it out. It's because you use `'%d'` for booleans not `'%b'`.
- This works, I can add users now.

## 21:18 - Next is adding the update part.
- For some reason it seems updating a record simply doesn't work if you try and it doesn't exist, it doesn't error out or anything, which is annoying.
- I'm going to have check the user exists first I guess.
- Okay, sorted, this is how it works:
```php
// Handle new subscriptions
if ($checkout->metadata->type == "membership") {
	$table_name = "user_payments";
	$user_hash = $checkout->client_reference_id;
	global $wpdb;

	$data = array(
		'user_hash' => $user_hash,
		'is_subscribed' => true,
	);
	$where = array(
		'user_hash' => $user_hash
	);
	$format = array('%s', '%d');
	$where_format = array('%s');


	$exists = $wpdb->get_var($wpdb->prepare("SELECT COUNT(*) FROM $table_name WHERE user_hash = %s", $user_hash));

	if ($exists) {
		$wpdb->update($table_name, $data, $where, $format, $where_format);
	} else {
		$wpdb->insert($table_name, $data, $format);
	}
}
```
- That's that complete, now I just need to do a basic paywall or something.
- So this is the last part required for the webhook in `functions.php`

## 22:06 - Setting up the Paywall on the post page
- I've changed the name of the table to `user_access` so that it can also contain how many free articles a user currently has access to.
- Okay, so I've got it set up now, I have the following function that checks for membership and updates the table of `user_access` to include IDs so that the search is faster:
```php
function has_membership($user_id)
{
	global $wpdb;
	$table_name = "user_access";

	// Check using the user's ID
	$subscribed = $wpdb->get_var($wpdb->prepare("SELECT is_subscribed FROM $table_name WHERE user_id = %d", $user_id));
	if (is_null($subscribed)) {
		$user_hash = get_user_hash($user_id);
		$subscribed = $wpdb->get_var($wpdb->prepare("SELECT is_subscribed FROM $table_name WHERE user_hash = %s", $user_hash));

		// Update the table to add or update the user
		if (is_null($subscribed)) {
			// Create a new user
			$data = array(
				"user_id" => $user_id,
				"user_hash" => $user_hash,
				"is_subscribed" => false
			);
			$format = array('%d', '%s', '%d');
			$wpdb->insert($table_name, $data, $format);
		} else {
			// Update a user to have an ID
			$data = array(
				"user_id" => $user_id,
			);
			$format = array('%d');
			$where = array(
				'user_hash' => $user_hash
			);
			$where_format = array('%s');
			$wpdb->update($table_name, $data, $where, $format, $where_format);
		}
	}

	if ($subscribed) {
		return true;
	} else {
		return false;
	}
}
```
- This function works great for making sure people have access to a post. By default, all posts are paywalled, although those that are tagged free will be allowed through.

# 23:10 - Making the paywall
- So I've spent a bit of time fiddling with chatgpt, CSS and html and I've come up with the following desing for the paywall
![[Pasted image 20240825231056.png]]
- Not bad eh! A nice white screen for now, it doesn't really match anything else, but oh well. It works.
- Anyway, this is what I had to do, go into `single.php` and find the location where:
```php
get_content();
```
- was being called.
- This is the location where the actual content of the post is loaded in. It's good to go into here and put in an `if` statement that says only let this be shown if the user has a membership.
- Then the rest was asking Chatgpt for some basic html and styling, which it duly gave me. Hell yeah!
- This will connect you up to the `/subscribe` endpoint that you have to click through to get to the Stripe checkout.
- This is the stuff I added into the `single.php` file:
```php
<?php
if ($membership) {
	// Show the content if the user has a membership
	the_content();
} else { ?>

	<div class="paywall-overlay">
		<div class="paywall-gradient"></div>
		<div class="paywall-content">
			<h2>This Content is Paywalled</h2>
			<p>To view this content, please subscribe to our service.</p>
			<a href="/subscribe" class="paywall-button">Subscribe Now</a>
		</div>
	</div>

<?php
	// Otherwise show the paywall
	$post = get_post(get_the_id());
	$content = $post->post_content;
	$sliced_content = substr($content, 0, 800);
	echo $sliced_content;
	echo "...";
}
?>
```
- Then this is the CSS that I used for the page:
```css
/* Basic styling for the post content */
.post-content {
  position: relative;
  padding: 20px;
  background-color: white;
  z-index: 1;
}

/* Paywall overlay styles */
.paywall-overlay {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 35%;
  background: rgba(255, 255, 255, 1); /* Semi-transparent black background */
  color: black;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  padding: 20px;
  z-index: 2; /* Ensure the overlay is on top */
  box-shadow: 0 -10px 20px rgba(139, 139, 139, 0.5); /* Slight shadow for pop effect */
}

/* Gradient to fade out text behind the overlay */
.paywall-gradient {
  position: absolute;
  top: -80px;
  left: 0;
  width: 100%;
  height: 80px;
  background: linear-gradient(to bottom, transparent, rgba(0, 0, 0, 0.7));
  z-index: 1;
}

/* Paywall content styling */
.paywall-content {
  max-width: 500px;
  padding: 20px;
  z-index: 2;
}

.paywall-content h2 {
  margin-bottom: 15px;
  font-size: 24px;
  font-weight: bold;
}

.paywall-content p {
  margin-bottom: 20px;
  font-size: 16px;
}

.paywall-button {
  display: inline-block;
  padding: 10px 20px;
  background-color: #e39b25;
  color: white;
  text-decoration: none;
  border-radius: 5px;
  font-size: 18px;
  transition: background-color 0.3s ease;
}

.paywall-button:hover {
  background-color: #e39b25;
}
```
- This does the trick!
- Now I just need to figure out how to make the content paywall thing nicer to look at etc.

## 23:22 - Wrapping up for the day
- I've just tested the flow from subscribe to stripe to checkout back to the original page showing that the `user_access` blocks you and sends only a small amount of content.
- All I need to do now is tidy everything up and put it into a format that can be used directly on the real site. Possibly adding `.env` file or something for tracking this.
- I also need the option to unsubscribe for the user, so account management or something.

# 2024-08-26
## 13:40 - Adding the .env file for swapping environments
- Okay, I've added a `.env` file, it has the variable LOCAL in it. Time to see if this works straight out of the box. This question helped me btw: https://stackoverflow.com/questions/67963371/load-a-env-file-with-php
- This didn't work when I was doing this:
```env
LOCAL=true
```
- But then it did work when I used this:
```env
LOCAL="true"
```
- Remember to use quotes!
- Anyway, I just tested this, it works great! The `.env` file is going to be really useful now I think.

## 14:08 - Round up of things to do
- [ ] Add the ability to access articles without membership, X per month.
- [ ] Add the ability to store subscription IDs for the current user.
- [ ] Add the ability for the user to cancel their subscription via Stripe.
- [ ] Add the ability to request a refund via Stripe.
- [ ] Add the ability to update someone's subscription when a refund is processed so that they don't have access.
- [ ] Add the ability to cancel someone's subscription when it next comes to an end via a webhook from stripe.
- [ ] Find a way to get the user to a stripe screen where they cancel which will then send out a webhook to `admin-post.php` to tell the server when a user has actually cancelled.
- [ ] Add the ability to store how many free articles a user has left this month.

# 2024-08-29
## 21:50 - Working on the ability to cancel your subscription
- Need to add a new page for cancelling, and then the stripe api call to tell them I've cancelled the subscription.
- Basically, I need to store the subscription that's attached to the current user and enforce that a user can only have a single subscription at a time. So they should be able to reach the subscribe page if they already have one.

## 22:41 - I've updated the `user_access` table to have a subscription ID
- This way it can take the ID returned by the stripe webhook.
- I've tested it using my dummy webhook and it works perfectly. #success
- I need to add a page for cancelling the stripe membership automatically, just like the one that sets up a checkout.

# 2024-08-29
## 09:41 - What I need to do
- Here's my goals for today:
	- [ ] Make the unsubscribe page.
	- [ ] Make the membership page.
	- [ ] Set up the Stripe Webhook that activates when someone's subscription ends.
	- [ ] Handle the Stripe Webhook for unsubscribing.

## 10:43 - Working on cancelling
- I've built a shit page for managing the subscription.
- Maybe it would be better to make this all done on Stripe or something?

## 10:53 - Simplifying loading an SQL table via the terminal #idea 
- Here's a cool idea, you can load a table into mysql with the following command:
```sh
mysql < your_table.sql
```
- But you can also go a big step further! You can set up a sequence of commands in the `.sql` file and simply drop the table from there instead. Here's my new file for setting up `user_access`:
```sql
use local;

drop table user_access;

CREATE TABLE user_access (
	id INT AUTO_INCREMENT PRIMARY KEY,
	user_id INT,
	user_hash VARCHAR(255) NOT NULL,
	is_subscribed BOOLEAN NOT NULL,
	subscription_id VARCHAR(255),
	stripe_customer_id VARCHAR(255),
	free_articles INT,
	free_date DATE
);
```
- This lets me load changes to the table with a single command, hell yeah!

# 2024-09-01
## 09:29 - Let's get it done!
- Goals:
	- [x] Integrate Stripe's membership page
	- [ ] Set up the Stripe Webhook that activates when someone's subscription ends.
	- [ ] Handle the Stripe Webhook for unsubscribing.

## 09:44 - Extending the `user_access` database
- This is the new database sql setup file:
```sql
use local;

drop table user_access;

CREATE TABLE user_access (
	id INT AUTO_INCREMENT PRIMARY KEY,
	user_id INT,
	user_hash VARCHAR(255) NOT NULL,
	is_subscribed BOOLEAN NOT NULL,
	subscription_id VARCHAR(255),
	stripe_customer_id VARCHAR(255),
	free_articles INT,
	free_date DATE,
	sub_ends DATE
);
```
- This adds a field for when a subscription ends. If the user is not subscribed this will be checked to see if they still have any subscription left.
- I've also added the `stripe_customer_id`, because this is required for automatically setting up the user on the stripe interface.

## 10:06 - Getting a 400 error
- I changed the name of the stripe action so had to change the url address, lol, whoops. #fix-found

## 10:10 - Getting redirected
- I've got a `.env` file that I'm trying to use to decide whether I'm running locally or not, I have it set to local at the moment, but it's not being read. Annoyingly I don't know how to see the running php process.
- You can find the error logs at this rough location in Local:
```sh
cd ~/Local Sites/site-name/logs/php/error.log
```
- Okay I was getting redirected again, and I figured out why. It was that pesky `wp_redirect` function making sure no-one can hit `admin-post.php` unless they're signed in, or they're calling the stripe webhook.
- With this altered I can now add back myself as a user again in `user_access`, which means I can access the Stripe Customer Portal:
![[Pasted image 20240901131615.png]]
- Baddingbing, baddaboom. Just the webhook now.

## 13:10 - Dummy the cancel webhook
- I'm back from church and fed. Time to make a dummy version of the webhook that cancels a subscription.
- Okay so, there are a few potential webhooks to look at for subscription management:

	- `customer.subscription.updated`
	- `customer.subscription.resumed`
	- `customer.subscription.pending_update_applied`
	- `customer.subscription.pending_update_expired`
	- `customer.subscription.paused`
	- `customer.subscription.deleted`

- I'm going to look at the `customer.subscription.updated` one because it seems to be used the most in the api calls I can see:
![[Pasted image 20240901133728.png]]

## 13:37 - Finding the right details in `customer.subscription.deleted
- I've changed my mind, I think the deleted one is probably easier. This is what stripe tells you to look at:
![[Pasted image 20240901135247.png]]
- https://dashboard.stripe.com/test/webhooks/create?endpoint_location=local
- Getting the Stripe CLI apparently lets you trigger events. So I may as well give that a go and see.
- Following the download the CLI link took me to here:
![[Pasted image 20240901135512.png]]
- Which let me have a crack at running my webhook anyway, without downloading anything. Ideal! #fix-found 
- I ran the following command:
```sh
stripe trigger customer.subscription.deleted
```
- annnnnd, I got this:
![[Pasted image 20240901135636.png]]
- Cool huh! Now I can use this as a template to create the dummy webhook that I want!

## 14:11 - Changes the Webhook makes
- Basically, because I know only deletion events of subscriptions are being sent to this endpoint the job is very simple:

	1. Verify the call is coming from Stripe.
	2. Once verified get the customer ID.
	3. Use the customer ID to update the user access table so that `is_subscribed = false`, and `subscription_id = null`.
	4. Arguably, I might want to say `customer_id` is null now as well. I'm not sure on this yet, it will probably become clear later.

#### #idea #aside
- Once I've got this done, I've basically finished everything apart from testing on the real site. I need to figure out how to test something like this in production smoothly. So that I can show it to the Mumie team.
- This is what's going to lead me into building the development site for showing progress etc.
- I need to do this using cloudfront or what have you pronto
- I just found out about awesome ways of taking notes as well! Really cool being able to do this sort of stuff in obsidian.

## 14:47 - I've finished the webhook
- The Webhook for the stripe subscription end is complete. Now I just need to make it possible for the url to be called without a redirect.
- This is what my stripe webhook looks like now:
```php
// Function to detect if the request is a Stripe webhook
function is_stripe_webhook()
{
	// Check if the action parameter is set and matches the one used for Stripe webhooks
	// Check if the script being executed is admin-post.php
	if (basename($_SERVER['SCRIPT_FILENAME']) === 'admin-post.php') {
		// Check if the action parameter is set and matches the one used for Stripe webhooks
		if (!empty($_REQUEST['action']) && $_REQUEST['action'] === 'stripe_payments') {
			return true;
		}

		if (!empty($_REQUEST['action']) && $_REQUEST['action'] === 'stripe_subscription_end') {
			return true;
		}
	}

	// Additional logic can be added here if you need to match other conditions
	// related to Stripe webhooks or other similar requests.

	return false;
}
```
- This works perfectly for letting me through the redirect block, but it doesn't update the database for some reason. I just need to figure out why now. #issue
- I've managed to find the fix by checking the `error.log` from before. It had the following:
```sh
[01-Sep-2024 13:49:23 UTC] WordPress database error Unknown column 'stripe_customer_id ' in 'where clause' for query UPDATE `user_access` SET `is_subscribed` = '', `subscription_id` = NULL WHERE `stripe_customer_id ` = 'cus_Qlh3ajkO7NQnZH' made by do_action('admin_post_nopriv_stripe_subscription_end'), WP_Hook->do_action, WP_Hook->apply_filters, handle_subscription_end
```
- This error shows that I forgot to delete the extra white space at the end of the `stripe_customer_id` column name. Whoops! Changing this fixed everything.
- Looks like it all works, although there is one important change to make on the customer id stuff:

>[!tip]
>Use the same `stripe_customer_id` the next time you set up a checkout for someone. That way all of their information can be managed in a single location.

- I'm going to highlight I think basically everything is done for Sarah and Laura and see when we can next meet to talk strategy.

## 16:00 - Implementing the tip above
- I've gone ahead and started adding the `customer_id` send, I may as well. It makes things better.
- But there's errors for some reason. 
- Okay, the error was with me doing it in an if statement or something, no idea why. I changed this and it's all fine. Whatever, don't care. Done now! #fix-found 

# 2024-09-03
## 19:10 - Writing up the paywall
- The paywall is complete now I just need to write up how I did it for posterity and such.
- https://docs.stripe.com/api

# 2024-09-09
## 23:45 - It's not over
- After discussion with the rest of the Mumie team there are still a couple of things to do and then we're going to deploy directly to the main site.
	- [ ] Add an option in the mumie dashboard for Sarah and Laura to add their Stripe account thing.
	- [ ] Write a tutorial explaining how this needs to be done.
	- [x] Figure out how to make products on Stripe.
	- [ ] Write a short tutorial on how to make products for Sarah and Laura.
	- [x] Figure out how to make coupons on Stripe.
	- [ ] Write a short tutorial on how to make products for Sarah and Laura.
	- [ ] Figure out how to pull the products from Stripe so that they'll be auto-displayed to the user when they attempt to subscribe.
	- [ ] Change the checkout page to use the product ID that they chose, rather than the price thing.
	- [ ] Change the appearance of the subscription page to look nicer and more in keeping with the Mumie aesthetic.
	- [ ] Change the appearance of the paywall to be more in keeping with the mumie aesthetic too.
	- [ ] Add the ability to give the user three free stories a month.
	- [ ] Add something that asks the user if they'd like to use one of their free stories a month. Or store this information somewhere obvious on the UI so it's clear that it's limited what they can see.

# 2024-09-10
## 23:34 - Adding a paywall
- I've looked at Stripe to figure out exactly how to create product catalogue line up and then create a coupon for that line up.
- I still need to check the following:
	- [ ] How do I read in the products via Stripe in WordPress?
	- [ ] What is the actual coupon code that we can give to people?
	- [ ] How can I make a coupon code that's 10% off and we give 10% automatically to the person who got the person to use that coupon code?
	- [ ] How do people put in a coupon at checkout? Does it automatically suggest it?


