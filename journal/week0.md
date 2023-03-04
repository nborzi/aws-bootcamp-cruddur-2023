# Week 0 — Billing and Architecture

#Created a Napkin version of the product crudder desired:

https://lucid.app/lucidchart/62e5833a-b62c-42a0-a1b9-7dc7a4cf20ad/view?invitationId=inv_4802c0d5-c793-4df8-85ec-d042f8642785&page=0_0#
![image](https://user-images.githubusercontent.com/86881008/222483922-b5c752e0-2e40-4822-a8eb-c5c407b6b200.png)


#Created a technical HLD of what tools are expected to be implemented in crudder
https://lucid.app/lucidchart/62e5833a-b62c-42a0-a1b9-7dc7a4cf20ad/view?invitationId=inv_4802c0d5-c793-4df8-85ec-d042f8642785&page=0_0#

![image](https://user-images.githubusercontent.com/86881008/222484008-e551bfeb-7a38-459c-bca5-9c389235b08c.png)

#Created Admin Account with billing privileges, also the billing set up and alarms ~(including cloudtrail and organization)

#Added CLI and some handy commands 
I did the following steps to install AWS CLI.
I installed the AWS CLI https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html AWS CLI Install Documentation Page

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

##Steps
#download
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
#unzip
unzip -u awscliv2.zip

#Run the install program.
sudo ./aws/install #or ./aws/install -i /usr/local/aws-cli -b /usr/local/bin

#Confirm the installation
aws --version
![image](https://user-images.githubusercontent.com/86881008/222907677-5f123b76-68d3-4ee5-9e25-e674a3c330ce.png)
