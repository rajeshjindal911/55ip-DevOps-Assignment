# 55ip-DevOps-Assignment
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
TasK-1: Description: A company is running application on AWS server. application server had crashed and the command to restart application server is not even working. The developer, being aware of some commands, tries to tweak around and discovers after running `df -h` that the server's disk is 100% full and most of the space is taken by /usr/src directory.

Solution:

cd /usr/src 
du -sh * -- wiil list out the file and directories with size. so will select the file and directory according with most size. we will delete/move that particular file or directory.
rm -rf dirname/filename

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Task-2: Description: There are 3 servers running App version 1(v1) in AWS under autoscaling.
Scaling policy is configured in such way that if average cpu percentage cross 60%, additional 2 new servers will get launched.
Now you have total 5 servers running v1. Team decide to deploy the new version of App version 2 ( v2) using ansible. you need to make sure your ansible inventory is upto date (with newly added servers)
Write a script that will create the ansible inventory with newly provisioned servers. Feel free mention any additional tool or different approach to achieve the same.

Solution:
Install pip,boto3 on ansible node.
download ec2.py and ec2.ini file for dynamic invenroty and move the files to /etc/ansible/
First we will create the ssh password-less connectivity between Ansible node and worker node.
Now run /ec2.py
you will be able to see the both the instance in ec2.py under ec2 group.
Now craete the AMI image of worker-node insatnce.
define the launch configuration with AMI image that we craeted in previous step and craete the ASG.
Now again come to Ansible node and again run /ec2.py. you will be able to see the newly created Server in ec2.py file under ec2 group.
You can check the connectivity of the servers with Ansible_node using follow command:
        ansible -i ec2.py ec2 -m ping 

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

task-3: Write a script to modify AWS security group to allow access to port 80 only from your public IP.
Solution:
#! /usr/local/bin/aws

read -p "enter the security_group id: " sg_id

IP=`curl -s http://whatismyip.akamai.com/`   -- to get the my_public_ip
ip_range=`aws ec2 describe-security-groups --group-ids $sg_id --output text | grep -A 1 "80" | grep -i IPRANGES | awk '{print $2}'`  --it will get the IP of already defined port 80
if [ -n $ip_range ]; then  --- condition is true if port is already defined
aws ec2 revoke-security-group-ingress --group-id $sg_id --protocol tcp --port 80 --cidr $ip_range  -- will delete the defined port 80 rule
aws ec2 authorize-security-group-ingress --group-id $sg_id --protocol tcp --port 80 --cidr $IP/32  --- will add port 80 rule with my_public_ip
else
aws ec2 authorize-security-group-ingress --group-id $sg_id --protocol tcp --port 80 --cidr $IP/32
fi

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
task-6: Write a script to identify AWS security credentials older than 90 day & renew them
Solution:

#! /usr/local/bin/aws

for user in $(aws iam list-users | grep -i UserName | awk -F':|,' '{print $2}' | tr -d '"' | tr -d ' ')  -- will list all the IAM users
do
creat_dat=`aws iam list-access-keys --user-name $user | grep -i CreateDate | awk '{print $2}' | tr -d '"' ` -- will retrive the date when the user was created
access_key=`aws iam list-access-keys --user-name $user | grep -i AccessKeyId | awk -F':|,' '{print $2}' | tr -d '"' | tr -d ' '`  --- will retrive the access_id of user
prof_age=`echo $(( ($(date +'%s') - $(date -d ${creat_dat} +'%s')) / 86400))`  -------will minus the user_created date from current date
if [ $prof_age -gt 90 ]; then   -- condition is true if user profile is more than 90 days old
   echo "$user was created $prof_age days ago. so it voilting secuity policy. it's credentials will be renewed"
   aws iam create-access-key --user-name $user 
   aws iam update-access-key --access-key-id $access_key --status Inactive --user-name $user    
   aws iam delete-access-key --access-key-id $access_key --user-name $user      ------- will delete the old access_keys
else
     echo "$user was created $prof_age days ago. it is as per the secuity policy"   --if user profile is less than 90 days old.
fi
done
