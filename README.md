# 55ip-DevOps-Assignment
TasK-1: Description: A company is running application on AWS server. application server had crashed and the command to restart application server is not even working. The developer, being aware of some commands, tries to tweak around and discovers after running `df -h` that the server's disk is 100% full and most of the space is taken by /usr/src directory.

Solution:

cd /usr/src 
du -sh * -- wiil list out the file and directories with size. so will select the file and directory according with most size. we will delete/move that particular file or directory.
rm -rf dirname/filename

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
