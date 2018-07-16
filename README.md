# CloudFormation-Workshop

# Exercise 1a - AutoScaling Creation Policy

In this exercise we will launch a sample template and observe the resource signals received as defined by the autoscaling creation policy:

- Launch template 'exercise-cfn-urf-autoscaling-policies.yaml' and select the 'SMALL' instance type
- Be sure to launch the instances in a *PUBLIC* subnet
- Observe the resource signals received in the CloudFormation events, eg:

``` 
19:12:11 UTC+0200	CREATE_COMPLETE	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	
19:12:10 UTC+0200	CREATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Received SUCCESS signal with UniqueId i-06f296d7c57a95051
19:12:10 UTC+0200	CREATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Received SUCCESS signal with UniqueId i-0e5e4c2d867cdb2ed
19:11:08 UTC+0200	CREATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Resource creation Initiated
19:11:07 UTC+0200	CREATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup
```

- Note that the created EC2 instances are t2.micro (small) instances

# Exercise 1b - AutoScaling Update Policy

In this exercise we will update the template, replacing the instances with t2.small (medium) instances types

- Select the previously created stack from exercise 1a in the AWS console and select 'Update Stack' > use current template
- Select a *MEDIUM* (t2.small) environment size
- Update the stack

This will cause a new Launch Configuration to be created following which AutoScaling will start to replace the instances 1 by 1 as defined in the batch size, eg:

```
19:22:42 UTC+0200	UPDATE_COMPLETE	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	
19:22:39 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Received SUCCESS signal with UniqueId i-06524248d6c52bbff
19:22:38 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	New instance(s) added to autoscaling group - Waiting on 1 resource signal(s) with a timeout of PT5M.
19:22:38 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Successfully terminated instance(s) [i-0e5e4c2d867cdb2ed] (Progress 100%).
19:21:45 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Terminating instance(s) [i-0e5e4c2d867cdb2ed]; replacing with 1 new instance(s).
19:21:44 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Received SUCCESS signal with UniqueId i-02d637fc0cdce81ae
19:21:43 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	New instance(s) added to autoscaling group - Waiting on 1 resource signal(s) with a timeout of PT5M.
19:21:43 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Successfully terminated instance(s) [i-06f296d7c57a95051] (Progress 50%).
19:20:50 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Terminating instance(s) [i-06f296d7c57a95051]; replacing with 1 new instance(s).
19:20:49 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Temporarily setting autoscaling group MinSize and DesiredCapacity to 2.
19:20:49 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Rolling update initiated. Terminating 2 obsolete instance(s) in batches of 1, while keeping at least 1 instance(s) in service. Waiting on resource signals with a timeout of PT5M when new instances are added to the autoscaling group.
19:20:46 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup
```

NOTE: At this point the autoscaling group instances will be t2.small (medium) as per the new launch configuration.

# Exercise 1c - Update Rollback Failed

In this exercise we will make and out of band change following which we will perform another stack update which will fail, causing the stack to be in a 'UPDATE_ROLLBACK_FAILED' state

## Create a new launch configuration

- In the AWS Console, copy the *launch configuration* created in exercise 1b
-- Be sure to copy the launch configuration, not *copy to launch template* 
-- Note the new launch configuration name, eg: excercise1-myLaunchConfig-28I80AK9E852Copy
- Accept the defaults & create the enw launc configration

## Out of band change

- Update the AutoScaling group with the new launch configuration created above
- Delete the previously CloudFormation created launch configuration (eg excercise1-myLaunchConfig-28I80AK9E852)

## Update the stack

- Select the stack in the AWS Console and select 'Update Stack' > use current template
- Select a *LARGE* (t2.medium) instance type
- Set *UpdateWithWaitCondition* to *true*
- Launch the stack

This time AutoScaling will again be performing a rolling update as a new launch configuration will be created
However! A new WaitCondition resource was introduced which requires resources signals (which won't send), thus the stack update will fail as the WaitCondition will fail to create successfully
At this point, the stack wil start to roll back, and reconfiguring the AutoScaling group with the previously created launch configuration (eg excercise1-myLaunchConfig-28I80AK9E852) *which we deleted*, this the stack rollback will fail.

```
19:40:44 UTC+0200	UPDATE_ROLLBACK_FAILED	AWS::CloudFormation::Stack	excercise1	The following resource(s) failed to update: [myAutoScalingGroup].
19:40:44 UTC+0200	UPDATE_FAILED	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	AutoScalingGroup's LaunchConfiguration excercise1-myLaunchConfig-28I80AK9E852 not found
19:40:44 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	
19:40:43 UTC+0200	UPDATE_COMPLETE	AWS::AutoScaling::LaunchConfiguration	myLaunchConfig	
19:40:40 UTC+0200	UPDATE_ROLLBACK_IN_PROGRESS	AWS::CloudFormation::Stack	excercise1	The following resource(s) failed to create: [myWaitCondition].
19:40:40 UTC+0200	CREATE_FAILED	AWS::CloudFormation::WaitCondition	myWaitCondition	Failed to receive 1 resource signal(s) within the specified duration
19:39:39 UTC+0200	CREATE_IN_PROGRESS	AWS::CloudFormation::WaitCondition	myWaitCondition	Resource creation Initiated
19:39:39 UTC+0200	CREATE_IN_PROGRESS	AWS::CloudFormation::WaitCondition	myWaitCondition	
19:39:37 UTC+0200	UPDATE_COMPLETE	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	
19:39:34 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Received SUCCESS signal with UniqueId i-0b4700a04d914f153
19:39:33 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	New instance(s) added to autoscaling group - Waiting on 1 resource signal(s) with a timeout of PT5M.
19:39:33 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Successfully terminated instance(s) [i-06524248d6c52bbff] (Progress 100%).
19:38:40 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Terminating instance(s) [i-06524248d6c52bbff]; replacing with 1 new instance(s).
19:38:40 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Received SUCCESS signal with UniqueId i-0c1e6ac662995f645
19:38:18 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	New instance(s) added to autoscaling group - Waiting on 1 resource signal(s) with a timeout of PT5M.
19:38:18 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Successfully terminated instance(s) [i-02d637fc0cdce81ae] (Progress 50%).
19:37:25 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Terminating instance(s) [i-02d637fc0cdce81ae]; replacing with 1 new instance(s).
19:37:24 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Temporarily setting autoscaling group MinSize and DesiredCapacity to 2.
19:37:24 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Rolling update initiated. Terminating 2 obsolete instance(s) in batches of 1, while keeping at least 1 instance(s) in service. Waiting on resource signals with a timeout of PT5M when new instances are added to the autoscaling group.
19:37:20 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup
```

# Recovering from the UPDATE_ROLLBACK_FAILED state

In order to recover from the UPDATE_ROLLBACK_FAILED state, we can recreate the launch configuration, giving it the same name which CloudFormation expects (excercise1-myLaunchConfig-28I80AK9E852)
Once done, we can perform a 'Continue Update Rollback' allowing the rollback to continue

NOTE: At this stage the instances are *LARGE* (t2.medium), thus the stack is now in an inconsistent state:
- The stack failed to update successfully (due to the failed waitcondition resource)
- And the stack also failed to roll back successfully (as the previous launch configation was deleted and no longer there)

NOTE: Notice that whilst the stack is in an UPDATE_ROLLBACK_FAILED state, we are unable to perform any further updates, the only options we have is to delete the stack altogether or choose 'CONTINUE_UPDATE_ROLLBACK'

Steps:

- Copy the launch configuration we manually created once more, giving it the name of the deleted launch configuration (eg excercise1-myLaunchConfig-28I80AK9E852)
- Once done, select the stack and choose 'Continue Update Rollback'

This will allow the rollback to continue, the AutoScaling Group will be updated with the previous launch configuration and the instances updated accordingly

Your stack should now be in an 'UPDATE_ROLLBACK_COMPLETE' state and the instances as they were before, MEDIUM (t2.small).

```
19:53:11 UTC+0200	UPDATE_COMPLETE	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	
19:53:08 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Received SUCCESS signal with UniqueId i-096621292ece00c70
19:52:46 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	New instance(s) added to autoscaling group - Waiting on 1 resource signal(s) with a timeout of PT5M.
19:52:46 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Successfully terminated instance(s) [i-0c1e6ac662995f645] (Progress 100%).
19:51:54 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Terminating instance(s) [i-0c1e6ac662995f645]; replacing with 1 new instance(s).
19:51:53 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Received SUCCESS signal with UniqueId i-094d05ff87d58ce6c
19:51:52 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	New instance(s) added to autoscaling group - Waiting on 1 resource signal(s) with a timeout of PT5M.
19:51:51 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Successfully terminated instance(s) [i-0b4700a04d914f153] (Progress 50%).
19:50:34 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Terminating instance(s) [i-0b4700a04d914f153]; replacing with 1 new instance(s).
19:50:33 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Temporarily setting autoscaling group MinSize and DesiredCapacity to 2.
19:50:33 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	Rolling update initiated. Terminating 2 obsolete instance(s) in batches of 1, while keeping at least 1 instance(s) in service. Waiting on resource signals with a timeout of PT5M when new instances are added to the autoscaling group.
19:50:30 UTC+0200	UPDATE_IN_PROGRESS	AWS::AutoScaling::AutoScalingGroup	myAutoScalingGroup	
19:50:28 UTC+0200	UPDATE_ROLLBACK_IN_PROGRESS	AWS::CloudFormation::Stack	excercise1	User Initiated
```


# Exercise 2 - UPDATE_ROLLBACK_FAILED (SKIPPING method)

In the previous exercise, when the stack was in an UPDATE_ROLLBACK_FAILED state, we could resolve the issue as we were able to recreate the deleted launch configuration with the same name and then selecting 'Continue Update Rollback'. However in some circumstances, this is not possible, such as for security groups. In this exercise we are going to demonstrate how to recover from such an event.

Also in contrast to the previous exercises where we updated the stack using the same template, but providing a different parameter (EnvironmentSize) which caused new resources to be created, in this exercise we are going to update the stack with separate templates.

## Launching the stack

- Launch the 'exercise-cfn-urf-instance-sg-create.yaml' template, selecting a subnet and VPC for your instance
- This stack will create an EC2 instance and security group which is referenced by the instance.

## Out of band change

- Edit the previously created instance and change it's security group - assign it the *default* security group for the VPC it was launched in
- *Delete* the security group that was created by CloudFormation during stack creation

## Update the stack

- Update the stack with template 'exercise-cfn-urf-instance-sg-update.1.yaml', leave the subnet and VPC as-is
- This stack differs from the previous template in that
    1. the 'SecurityGroupIds' property is commented out (thus will be removed from the instance), and a WaitCondition
    2. a 'WaitCondition' resource is added which will cause our stack update to fail

Note the 'Modify' action for the instance and the 'Add' action for the new WaitCondition resource when previewing the stack changes:

```
Preview your changes
Based on your input, CloudFormation will change the following resources. For more information, choose View change set details.

Action	Logical ID	Physical ID	Resource type	Replacement
Modify	Instance	i-08ae03922304d1e71	AWS::EC2::Instance	Conditional
Add	WaitCondition		AWS::CloudFormation::WaitCondition
```

The stack should now again be in an UPDATE_ROLLBACK_FAILED state, as:

- The instance was updated (the security group was removed), however creation of the subsequent resource (WaitConditionHandle) failed, causing the stack to roll back
- However during the stack rollback, CloudFormation was unable to configure the instance with the previous security group (as it no longer exists)

```
20:23:48 UTC+0200	UPDATE_ROLLBACK_FAILED	AWS::CloudFormation::Stack	exercise2	The following resource(s) failed to update: [Instance].
20:23:47 UTC+0200	UPDATE_FAILED	AWS::EC2::Instance	Instance	The security group 'sg-061e3878940415b63' does not exist (Service: AmazonEC2; Status Code: 400; Error Code: InvalidGroup.NotFound; Request ID: a578d3c8-c539-4f2c-ba58-058df13b8658)
20:23:46 UTC+0200	UPDATE_IN_PROGRESS	AWS::EC2::Instance	Instance	
20:23:42 UTC+0200	UPDATE_ROLLBACK_IN_PROGRESS	AWS::CloudFormation::Stack	exercise2	The following resource(s) failed to create: [WaitCondition].
20:23:41 UTC+0200	CREATE_FAILED	AWS::CloudFormation::WaitCondition	WaitCondition	Failed to receive 1 resource signal(s) within the specified duration
20:22:40 UTC+0200	CREATE_IN_PROGRESS	AWS::CloudFormation::WaitCondition	WaitCondition	Resource creation Initiated
20:22:40 UTC+0200	CREATE_IN_PROGRESS	AWS::CloudFormation::WaitCondition	WaitCondition	
20:22:38 UTC+0200	UPDATE_COMPLETE	AWS::EC2::Instance	Instance	
20:22:22 UTC+0200	UPDATE_IN_PROGRESS	AWS::EC2::Instance	Instance	
20:22:18 UTC+0200	UPDATE_IN_PROGRESS	AWS::CloudFormation::Stack	exercise2	User Initiated
```

## Recovering from the situation

The stack is now in an UPDATE_ROLLBACK_FAILED state, however we have a problem as we will not be able to recreate a security group with a specific ID (eg sg-061e3878940415b63) as these are auto-assigned. 

To recover from this, we are going to issue and 'Continue Update Rollback' as before, however this time we will be skipping the failed resource (EC instance). 

- Select the stack and then choose 'Continue Update Rollback', however this time in the advanced section enter the logical ID of the resource to be skipped during rollback. 
- In this case the logicalID is simply 'instance' which is the name we gave it in the stack.

Once done, the stack will again be in an UPDATE_ROLLBACK_COMPLETE STATE:

```
20:30:45 UTC+0200	UPDATE_ROLLBACK_COMPLETE	AWS::CloudFormation::Stack	exercise2	
20:30:45 UTC+0200	DELETE_COMPLETE	AWS::CloudFormation::WaitCondition	WaitCondition	
20:30:44 UTC+0200	DELETE_IN_PROGRESS	AWS::CloudFormation::WaitCondition	WaitCondition	
20:30:43 UTC+0200	UPDATE_ROLLBACK_COMPLETE_CLEANUP_IN_PROGRESS	AWS::CloudFormation::Stack	exercise2	
20:30:42 UTC+0200	UPDATE_COMPLETE	AWS::EC2::Instance	Instance	Resource skipped during UpdateRollback
20:30:40 UTC+0200	UPDATE_ROLLBACK_IN_PROGRESS	AWS::CloudFormation::Stack	exercise2	User Initiated

```

However this time, the stack is in an inconsistent state and does not match reality. As we are unable to recreate a security group with a specific id, we are going to be perform a 2nd stack update using 'exercise-cfn-urf-instance-sg-update.2.yaml'

Steps: 

- Update the stack with template 'exercise-cfn-urf-instance-sg-update.2.yaml'
- Leave the subnet and VPC as-is

In the 'exercise-cfn-urf-instance-sg-create.yaml' stack, the security group was called 'InstanceSecurityGroup', however this resource was since deleted due to an out of band change. In the 'exercise-cfn-urf-instance-sg-update.2.yaml' stack, the security group was renamed and called 'InstanceSecurityGroup2'. This will cause the stack to:

- Remove the previously created security group
- Create a new security group as per the stack changes, for example:

```
Preview your changes
Based on your input, CloudFormation will change the following resources. For more information, choose View change set details.

Action	Logical ID	Physical ID	Resource type	Replacement
Modify	Instance	i-08ae03922304d1e71	AWS::EC2::Instance	Conditional
Remove	InstanceSecurityGroup	sg-061e3878940415b63	AWS::EC2::SecurityGroup	
Add	InstanceSecurityGroup2		AWS::EC2::SecurityGroup
```

If everything went well, the stack will again be in an 'UPDATE_COMPLETE' state

```
20:38:21 UTC+0200	UPDATE_COMPLETE	AWS::CloudFormation::Stack	exercise2	
20:38:21 UTC+0200	DELETE_COMPLETE	AWS::EC2::SecurityGroup	InstanceSecurityGroup	
20:38:20 UTC+0200	DELETE_IN_PROGRESS	AWS::EC2::SecurityGroup	InstanceSecurityGroup	
Physical ID:sg-061e3878940415b63
Client Request Token:Console-UpdateStack-b03e29a5-0bfb-4d55-bc48-6e5a1a34bd1a
20:38:18 UTC+0200	UPDATE_COMPLETE_CLEANUP_IN_PROGRESS	AWS::CloudFormation::Stack	exercise2	
20:38:17 UTC+0200	UPDATE_COMPLETE	AWS::EC2::Instance	Instance	
20:38:01 UTC+0200	UPDATE_IN_PROGRESS	AWS::EC2::Instance	Instance	
20:37:58 UTC+0200	CREATE_COMPLETE	AWS::EC2::SecurityGroup	InstanceSecurityGroup2	
Physical ID:sg-09aad7ab8aff32c6d
Client Request Token:Console-UpdateStack-b03e29a5-0bfb-4d55-bc48-6e5a1a34bd1a
20:37:57 UTC+0200	CREATE_IN_PROGRESS	AWS::EC2::SecurityGroup	InstanceSecurityGroup2	Resource creation Initiated
20:37:56 UTC+0200	CREATE_IN_PROGRESS	AWS::EC2::SecurityGroup	InstanceSecurityGroup2	
20:37:53 UTC+0200	UPDATE_IN_PROGRESS	AWS::CloudFormation::Stack	exercise2	User Initiated
```
