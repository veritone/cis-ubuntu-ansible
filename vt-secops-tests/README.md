# Running CIS Hardening Monthly

I believe we are just in an OK spot with this for now. I really think we need to replace the Ansible role with something else to move forward by leaps and bounds. It is just hard to allocate the time necessary to do that when we have a tool that works to a degree.

## Role Configuration

See **roles/cis/defaults/main.yml** under the role source code.

The original auther of the role did a decent job of commenting this file so you know which variables belong to which role. You can always reverse this by looking at a tasks file to find the variable and using it to search the defaults file.

### Veritone Custom Variables

There were a few rules that caused us a great deal of angst with Packer and hardening leading up to FedRAMP. To get through FedRAMP, I hacked the file to comment out those rules.

For the monthly runs, I have added custom variables to cover those rules. This way you can flip a boolean to enable/disable a rule vs. commenting out a block of code.

Those variables are:

```yaml
vt_control_5_2_7: False
vt_control_5_2_9: False
vt_control_5_2_14: True
```

The rules associated with 5.2.7 and 5.2.9 are anathema to our use of Packer and our use of immutable infrastructure. I don't see us ever being compliant with those rules.

We can use 5.2.14 to our advantage. All of the user and group lists in the **roles/cis/defaults/main.yml** file are empty to start. We can then add the Packer user to the users list, enable this rule and only Packer will be able to access the instance.

## Vagrant SSH Test

The number one problem we had when trying to integrate hardening into the Packer builds, was cutting of Packer's SSH access.

This README.md lives in **cis-ubuntu-ansible/vt-secops-tests**. I sense Gary will want that to read infosec as he uses that term for us everywhere. We can fix that as part of the PR.

That directory contains all you need to do an SSH test.

```
cd vt-secops-tests
vagrant up
vagrant ssh
```

If hardening has broken SSH, you won't be able to SSH as the vagrant user which means Packer won't be able to SSH either.

We will have a much friendlier relationshiop with DevOps if we consistently give them hardening that Packer can access. It doesn't mean the hardening will be perfect out of the box, but at least they won't hit a basic issue to start.

## Other Tests

There are a lot of configuration options in **roles/cis/defaults/main.yml**. This means there are plenty of things you can tweak each month. The risk you run is these are things we won't see with an SSH test. These are the kinds of things the application code will break on.

At some point, we might want to see if we could make this more valuable by testing against n number of applications. I know we don't have a test team at Veritone, yet. When we do, we should probably talk to them about automated testing of the hardening.

What I envision is a smoke test per application. We would stand an application up on the newest hardened AMI, run the smoke test and see what breaks.

