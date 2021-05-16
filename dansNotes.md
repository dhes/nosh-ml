1) Don't include this (dansNotes.md) file in pull requests. 
2) Remember to remove the hard-coded mysql password in database.php
3) In composer.json the local file repository ../tcpdi-merger should be used - if you can do it - for development, because github complains if we touch the remote repository too often. For the pull request it should be https://github.com/dhes/tcpdi-merger.
PDF version of HPQ questionnaire at https://www.hbdclinic.com/pdfs/Toxicity%20and%20Inflammation.pdf.

4) To Controller.form_users after line 9611 I have added something like:
            $items[] = [
                'name' => 'username',
                'label' => trans('noshform.username'),
                'type' => 'text',
                'required' => true,
                'readonly' => true,
                'default_value' => $data['username']
            ];

            $items[] = [
                'name' => 'password',
                'label' => trans('noshform.admin_password'), #probably noshform.new_password
                'type' => 'password',
                'required' => true,
                'default_value' => null
            ];
            $items[] = [
                'name' => 'confirm_password',
                'label' => trans('noshform.confirm_password'),
                'type' => 'password',
                'required' => true,
                'default_value' => null
            ];
...to allow admin to create a password for new user. The section is marked with my initials in comments. 

5) Success! Fields added to form! Now write the processing logic. Look to InstallController around line 505 (where it hashes the password).

6) A nice article on docker networks: https://docs.docker.com/network/network-tutorial-standalone/

7) Info from add physician user for mailgun troubleshooting:
NOTICE: PHP message: dan@heslinga.us
NOTICE: PHP message: Manoa Innovations
NOTICE: PHP message: template: auth.emails.generic
NOTICE: PHP message: to: heslingamd@icloud.com
NOTICE: PHP message: data message: 1
NOTICE: PHP message: to: heslingamd@icloud.com
NOTICE: PHP message: subject: Invitation to NOSH ChartingSystem
NOTICE: PHP message: MAIL_DRIVER=mailgun: mailgun
NOTICE: PHP message: MAIL_HOST=smtp.mailgun.org: 
NOTICE: PHP message: MAIL_PORT=587: // DH not set even though in mail.php
NOTICE: PHP message: MAIL_USERNAME=postmaster@sandboxcc*****************.mailgun.org: // DH blank
NOTICE: PHP message: MAIL_PASSWORD=******************: // DH blank
NOTICE: PHP message: MAIL_ENCRYPTION=tls: // DH blank, is also is set in mail.php
NOTICE: PHP message: Exception thrown in /var/www/nosh/vendor/guzzlehttp/guzzle/src/Exception/RequestException.php on line 113: [Code 401]
                Client error: `POST https://api.mailgun.net/v3/mg.heslingamd.com/messages.mime` resulted in a `401 UNAUTHORIZED` response:
Forbidden
Debug code: 
    after        if (env('MAIL_DRIVER') !== 'none') {
            $from_email = $practice->email;  // admin email
            $from_name = $practice->practice_name; // Manoa innovations:

            error_log('from_email: ' . $from_email); //DH
            error_log('from_name: ' . $from_name); //DH start
            error_log('template: ' . $template);
            error_log('to: ' . $to);
            error_log('data message: ' . print_r($data_message));
            error_log('to: ' . $to);
            error_log('subject: ' . $subject);
            error_log('MAIL_DRIVER=mailgun: ' . env('MAIL_DRIVER'));
            error_log('MAIL_HOST=smtp.mailgun.org: ' . env('MAIL_HOST'));
            error_log('MAIL_PORT=587: ' . env('MAIL_PORT='));
            error_log('MAIL_USERNAME=postmaster@sandboxcc*****************.mailgun.org: ' . env('MAIL_USERNAME'));
            error_log('MAIL_PASSWORD=******************: ' . env('MAIL_PASSWORD'));
            error_log('MAIL_ENCRYPTION=tls: ' . env('MAIL_ENCRYPTION')); // DH end

in exception block: 
                $errorCode = $e->getCode();
                $errorMessage = $e->getMessage();
                $errorFile = $e->getFile();
                $errorLine = $e->getLine();
                error_log("Exception thrown in $errorFile on line $errorLine: [Code $errorCode]
                $errorMessage");
                // DH end

8) The instructions on the web site for setting up on godaddy are not right. This video lays it out nicely; 

https://help.gohighlevel.com/support/solutions/articles/48000981678-mailgun-setup-godaddy-domain-setup

Basically, leave off the base domain name (heslingamd.com in my case) in the host setting i.e. "mg" not "mg.heslingamd.com"

8) overview of adding and mainting alpine packages. which 
9) This worked: 
# Swaks is an smtp of CURL, install it first:
curl http://www.jetmore.org/john/code/swaks/files/swaks-20130209.0/swaks -o swaks
# Set the permissions for the script so you can run it
chmod +x swaks
# It's based on perl, so install perl
sudo apt-get -y install perl //apk add perl on alpine machines
# now send!
./swaks --auth \
        --server smtp.mailgun.org \
        --au postmaster@mg.heslingamd.com \
        --ap 9cbe59c82e62a949239a74d24ab07249-a09d6718-84eb2267 \
        --to heslingamd@icloud.com \
        --h-Subject: "hello" \
        --body 'Testing some Mailgun awesomeness!'

Testing API method:

curl -s --user 'api:1ced6854c7d3cf4facdf15f7c13651d0-8889127d-fdae178d' \
 https://api.mailgun.net/v3/mg.heslingamd.com/messages \
 -F from='Excited User <mailgun@mg.heslingamd.com>' \
 -F to=heslingamd@icloud.com \
 -F subject='Hello' \
 -F text='Testing some Mailgun awesomeness!'

 10) Tricks and gotchas in Nosh Mailgun setup. 

 - A working Mailgun setup is essential for any use of Nosh. The workflow for adding accounts for staff such as doctors, assistants and billers require that Nosh's built-in email is activated. New users are created by the administrator. The admin person enters just a first name, last name and email address and saves this information. Nosh automatically send an invitation email to the new user saying "You are invited to use the NOSH ChartingSystem for Your Practice. Go to http://localhost/accept_invitation/veryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryLongString to get registered." This leads the new user to a Nosh setup screen where they set a username and password for themselves. 

This is a good practice. The admin should not know the usernames and password of other users, which is what would happen if the admin selected usernames and passwords. On the other hand it requires Nosh's internal main system to be setup correctly. To do this you have to understand (or learn) at least the basic of DNS records and the Mailgun system. If you do proceed along those lines, please read on. 

 - The instructions for setting up Mailgun for a GoDaddy domain are misleading. You set up your Mailgun account on the Mailgun web site, but you have to go you your domain hosting company's web site to set up the DNS records. This is GoDaddy in my case. I've done this a handful of times and it always give me the creeps. I use my domain for all of my email. What if I erase an essential record in my GoDaddy DNS and mess up all my email? I've gotting more comfortable with it recently. Just be sure to only ADD records to your DNS file, and follow the instructions carefully. 

  - The problem is in Mailgun's instructions on the DNS setup. Let's say you domain is yourdomain.com and you decide that your Mailgun domain will be mg.yourdomain.com. The Mailgun web site shows you how to add text records and MX records to yourDNS record (at GoDaddy). However, wherever Mailgun instructs you to use mg.yourdomain.com in the host field, you should acutally use just mg. Not the full domain name. If you use the full domain name then Mailgun verification will fail. For a nice video, see link above where the instructed actually sets up the DNS records on GoDaddy while you watch. 

  - To be fair to Mailgun, they have an intuitive section that show syou how to test your setup using curl and perl from the command line. I was able to figure out the Nosh setup using those examples. 
 
  - I'm not sure about other domain hosting companies. I only have experience with GoDaddy. 

 - Nosh uses the API method to access Mailgun. Not smtp. Don't be fooled by 'driver' => env('MAIL_DRIVER', 'smtp') and 'host' => env('MAIL_HOST', 'smtp.mailgun.org') in config/mail.php. 

 - Accordingly, all of the following values in nosh\env (in docker-nosh) should remain null (the default in docker-nosh), as shown:

    MAIL_HOST=null
    MAIL_PORT=null
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null

 - the mailgun_domain is set in mailgun_domain.txt (in docker-nosh). Here you do put the full domain name, e.g. mg.yourdomain.com. 

 - The mailgun_secret (set in mailgun_secret.txt) is the API key set by Mailgun. Not the stmp password, as I initially assumed (see above). The Mailgun web site will give you an API key after you succesfully set up your DNS record and verify it on the Mailgun web site. Copy that API and paste it into mailgun_secret.txt. Then restart your server and give it a try!

11) A nice overview of grants in mysql: https://www.mysqltutorial.org/mysql-grant.aspx

12) A nice overview of the bash if command. Dr. Chen uses this in his bash script. 

13) Think about a way to allow asuser to create and access a new database without granting all privleges. 

14) Oh boy! A Forbes story on Epic and Judy Faulker.
    https://www.forbes.com/sites/katiejennings/2021/04/08/billionaire-judy-faulkner-epic-systems/?sh=20bc5730575a
15) demographics_ table all have pid of int type whereas demographics pid is bigint. Fix with:
/*alter table demographics_notes modify column pid bigint*/
/*alter table demographics_plus modify column pid bigint*/
/*alter table demographics_relate modify column pid bigint*/
I added a few lines to grant.sql to automate this. 

16) nosh doesn't like m and f for gender. The selections are male female individual. Do: /*update demographics set sex = 'm' where sex = 'M'*/ and similar for F and U
and:
/*update demographics set sex = 'f' where sex = 'F'*/
/*update demographics set sex = 'u' where sex = 'U'*/
to find items with caps use select sex from demographics where BINARY sex='M'
17) Nosh doesn't like USA; it wants United Stats. It prefers HI to Hawaii (in the demographics table.) Nosh uses the https://packalyst.com/packages/package/pragmarx/countries as shown by the use PragmaRX\Countries\Package\Countries; line in Controller.php. It looks up states in countries and uses that to populate the state field in demographics. With USA in the database it will throw an error. It's good with 'United States' and 'Hawaii'
Cleaning up the countries after importing to nosh requires the following lines:
/*select distinct country, count(country) from demographics d3 group by country ORDER by count(country) DESC */
/*update demographics set country ='United States' where country='USA'*/
/*update demographics set country ='United States' where country='US'*/
/*update demographics set country ='United States' where country is NULL*/
/*update demographics set country ='Canada' where country = 'Can'*/
/*update demographics set country ='Japan' where country = 'JAP'*/
/*update demographics set country ='United Kingdom' where country = 'England'*/
/*update demographics set country ='Germany' where country = 'GER'*/
/*update demographics set country ='Marshall Islands' where country = 'Rep. of Marshall Islands'*/
/*Now for all the rest*/
/*update demographics set country = 'United States' where country not in ('Germany', 'United States', 'Australia', 'Canada', 'Japan', 'Norway', 'Australia', 'United Kingdom', 'Marshall Islands')*/
With all of the cleanups the demographics import pretty well and create charts in Nosh!
17) Next - problems lists! Start with the problem table in psql and map it to issues with type problem list. We'll put in a couple of problems for ferd berfel as an example.

18) NDC codes in Logician don't seem to map to RxNorm codes in Nosh. It looks like the old Logician MedInfo from FirstData has been rebranded to FDB Medknowledge. No publicly accesible. 
Crosswalk example generic lisinopril 10 mg Nosh: 00093111310 Logician: 67544015958. These two codes are found in rxnsat.rrf as: 


    314076|||9792290|AUI|000390|||NDC|NDDF|67544015958|O|| // with nddf in field ten
    314076|||12339748|AUI|314076|||NDC|RXNORM|00093111310|N|4096|

If the last field contains 4096 it is in the Current Prescribable Content. This suggest a crosswalk from Logician to Nosh: 

    -locate the Logician (NDFF) number
    -find the corresponding row that starts with the same id in the first column and has 4096 in the last column (col 11). Use the value in col 9 from that row. 

In field 10 the value NDDF designiates FDB MedKnowledge (formerly NDDF Plus). That might be the logician number. VANDF is the Veterans Health Administration National Drug File. 

