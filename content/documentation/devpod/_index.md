---
title: "DevPod on AWS"
date: 2025-02-11T20:00:00+02:00
weight: 1
cascade:
  type: docs
---

## Sources

- [DevPod](https://devpod.sh/docs/what-is-devpod) ⚙️
- [DevContainer](https://containers.dev) 🐳

## Introduction 🚀

In this article, I’d like to showcase a fantastic tool that sits in the same family as GitPod and Codespaces: **DevPod**! It empowers you to create development environments effortlessly—without getting locked into any particular vendor. 🔒❌

DevPod is based on **DevContainer** architecture and uses the specs found in a [devcontainer.json](https://containers.dev) file to spin up your dev setup. Personally, I love it for quickly and reliably deploying Network Labs with ContainerLab. 💻🧰

## What a DevContainer Is 🤔

A development container (often called a “dev container”) lets you use a container as a full-featured dev environment. (Check out the official [containers.dev documentation](https://containers.dev) for more details.)

Let’s break it down:

Have you ever heard the phrase “It works on my machine”? If you’re a developer, you’ve probably run headfirst into this problem when collaborating with others. But guess what? **DevContainers** solve the mystery of environment drift by providing consistent, reproducible Docker-based environments. Bye-bye setup headaches! 💆‍♀️

Using DevContainers, you just need:

1. A `.devcontainer` folder in your project.
2. A `devcontainer.json` file that tells VS Code how to configure the container.
3. Docker installed locally

The backbone of the DevContainer is the `devcontainer.json` file. For example:

```json
{
  "name": "Python DevContainer",
  "image": "mcr.microsoft.com/devcontainers/python:3.10",
  "features": {
    "docker-in-docker": "latest"
  },
  "extensions": [
    "ms-python.python",
    "ms-azuretools.vscode-docker"
  ]
}
```

**What’s happening here?** 🕵️‍♀️

- **"image"** – This uses a Python 3.10 environment that’s ready to go.
- **"features"** – This adds Docker support inside the container.
- **"extensions"** – Installs useful Python and Docker extensions in VS Code.

DevContainers are awesome for local dev, but sometimes you need more muscle—maybe your workloads are huge, or you want to run specialized labs. That’s where **DevPod** comes in. 💪  
**To be able to deployed lab easily on Cloud**

## What DevPod Is 🤖

**DevPod** is an open-source tool that lets you spin up dev environments either on your local machine or in the cloud of your choice. Imagine a self-hosted, super-customizable version of GitHub Codespaces. 🎉

In my day-to-day networking escapades, I deploy ContainerLab-based setups with DevContainers on AWS. Let’s see how you can use DevPod to do exactly that (ContainerLab details will follow in another post, I promise!). 😜

## AWS Provider 🌐

DevPod uses **Providers**, which are like configuration modules that define where and how DevPod launches your environment. Here’s the Provider list:

![Provider_list](provider.png#center)

We’ll focus on the **AWS Provider**—though there are many config options:

![Provider_list](aws_options.png#center)

Before you freak out at all those toggles, don’t worry. If it’s just you tinkering, the defaults are usually fine. 🙌

> [!NOTE] **Open Source Perks** 🎁
>
> Being open-source means you can pop the hood and see exactly how DevPod works. Check out the AWS code [here](https://github.com/loft-sh/devpod-provider-aws/tree/main) if you’re curious.

## How the AWS Code Works

Let’s break down what DevPod will do on AWS by looking at the [aws.go code](https://github.com/loft-sh/devpod-provider-aws/blob/main/pkg/aws/aws.go). From a high-level, it handles:

1. **Initialization**: Reading configuration and setting up AWS SDK clients (with custom credentials if needed).
2. **Networking**: Finding or creating the appropriate subnet, VPC, and security groups.
3. **AMI Selection**: Choosing a suitable AMI (defaulting to a recent Ubuntu 22.04 image) and determining the root device.
4. **IAM Setup**: Ensuring an appropriate instance role and instance profile exist, complete with policies.
5. **Instance Lifecycle**: Creating, starting, stopping, checking status, and deleting instances.
6. **User Data Injection**: Generating a script (embedded as Base64) that configures the instance (sets up users and SSH keys) on first boot.
7. **Optional DNS**: Managing Route 53 records for the instance if the configuration calls for it.

From my perspective, two points stand out as potentially the most critical:

- **(#4) IAM Setup**
- **(#6) User Data Injection**

### Why are #4 and #6 “Tricky”?

- **IAM Setup** is primarily handled by the `CreateDevpodInstanceProfile` function. It creates a role named `devpod-ec2-role` that can do the following:
  - **EC2 Operations**: For example, it can describe or stop instances—particularly the instance associated with itself.
  - **SSM Operations**: By attaching the AmazonSSMManagedInstanceCore policy, the instance can be managed by AWS Systems Manager.
  - **KMS Operations (Optional)**: If configured, it can perform `kms:Decrypt` on a specific KMS key, helpful for handling session data or secrets.

- **User Data Injection** is basically a startup script inserted into the instance as Base64. That script sets up a `devpod` user with sudo rights, creates SSH folders, and plops your public key in place. In the code, [it looks like](https://github.com/loft-sh/devpod-provider-aws/blob/9d2730c34ecee40cb42596c602381b92ad9c6682/pkg/aws/aws.go#L967-L980):

```bash
useradd devpod -d /home/devpod
mkdir -p /home/devpod
if grep -q sudo /etc/groups; then
  usermod -aG sudo devpod
elif grep -q wheel /etc/groups; then
  usermod -aG wheel devpod
fi

echo "devpod ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/91-devpod
mkdir -p /home/devpod/.ssh
echo " + string(publicKey) + " >> /home/devpod/.ssh/authorized_keys
chmod 0700 /home/devpod/.ssh
chmod 0600 /home/devpod/.ssh/authorized_keys
chown -R devpod:devpod /home/devpod
```

Since DevPod is open source, you can easily check this out yourself. It’s a great learning tool if you’re curious about the nuts and bolts! 🔩

## IAM Roles and Policies

You’ll need to set up an IAM user and attach an IAM policy that grants just enough permissions for DevPod. For example:

- **EC2 Actions:**
  - Describe: `ec2:DescribeSubnets`, `ec2:DescribeVpcs`, `ec2:DescribeImages`, `ec2:DescribeInstances`, `ec2:DescribeSecurityGroups`
  - Manage Instances: `ec2:RunInstances`, `ec2:StartInstances`, `ec2:StopInstances`, `ec2:TerminateInstances`, `ec2:CancelSpotInstanceRequests`
  - Security Groups & Tags: `ec2:CreateSecurityGroup`, `ec2:AuthorizeSecurityGroupIngress`, `ec2:CreateTags`
- **IAM Actions:**
  - `iam:GetInstanceProfile`, `iam:CreateRole`, `iam:PutRolePolicy`, `iam:AttachRolePolicy`, `iam:CreateInstanceProfile`, `iam:AddRoleToInstanceProfile`
- **Route 53 (Optional):**
  - `route53:ListHostedZones`, `route53:GetHostedZone`, `route53:ChangeResourceRecordSets`

## AWS Configuration 🏗️

I typically use the AWS web console to set this up, but you can absolutely do it all via CLI.

### Step 1: Log in to the AWS Console

1. Head to [AWS Management Console](https://aws.amazon.com/console/).
2. Use an account with the proper rights to create IAM resources.

### Step 2: Create a Custom IAM Policy

#### **A. Go to the IAM Console**

- In the AWS menu, find **IAM**.

#### **B. Create a New Policy**

1. Click **Policies** on the left.
2. Press **Create policy**.

#### **C. Switch to the JSON Tab**

- Paste in something like this, adjusting if needed:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EC2Actions",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeSubnets",
        "ec2:DescribeVpcs",
        "ec2:DescribeImages",
        "ec2:DescribeInstances",
        "ec2:DescribeSecurityGroups",
        "ec2:RunInstances",
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:TerminateInstances",
        "ec2:CancelSpotInstanceRequests",
        "ec2:CreateSecurityGroup",
        "ec2:AuthorizeSecurityGroupIngress",
        "ec2:CreateTags",
        "ec2:DeleteTags"
      ],
      "Resource": "*"
    },
    {
      "Sid": "IAMActions",
      "Effect": "Allow",
      "Action": [
        "iam:GetInstanceProfile",
        "iam:CreateRole",
        "iam:PutRolePolicy",
        "iam:AttachRolePolicy",
        "iam:CreateInstanceProfile",
        "iam:AddRoleToInstanceProfile"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Route53Actions",
      "Effect": "Allow",
      "Action": [
        "route53:ListHostedZones",
        "route53:GetHostedZone",
        "route53:ChangeResourceRecordSets"
      ],
      "Resource": "*"
    }
  ]
}
```

- Click **Next** (tags optional), then **Next: Review**.

#### **D. Review and Create**

1. Name it `DevpodToolPolicy` (or whatever you like).
2. Optional description.
3. Click **Create policy**.

### Step 3: Create/Update the IAM User

#### **A. Create a New User (If Needed)**

1. Click **Users** in IAM.
2. **Add user**.
3. Name it (e.g., `devpod-tool-user`).
4. Choose **Programmatic access** if you want CLI access. 🤖
5. **Next**.

#### **B. Attach Your New Policy**

1. On the permissions page, pick **Attach policies directly**.
2. Check `DevpodToolPolicy`.
3. Click **Create**.
4. That’s it!

### Step 4: Verify & Done

Head back to **Users** → **devpod-tool-user** → **Permissions** to confirm `DevpodToolPolicy` is attached. ✅

### Step 5: Use Those Credentials

- If you created a programmatic user, make sure you note the **Access Key ID** and **Secret Access Key**.

![devpod_user_sumup](devpod_user.png#center)

**Bonus**: Keep track of your **VPC ID** (under the VPC service in AWS). You’ll need it when setting up DevPod.

## Configure DevPod 🛠️

### 1. Configure AWS Profile

```bash
aws configure --profile Devpod
```

When prompted:

1. Access Key ID
2. Secret Access Key
3. Default region (e.g., `eu-west-1`)
4. Output format (I usually leave it blank.)

### 2. Add Profile to DevPod

1. In DevPod, create a new provider and pick **AWS**.
2. Select the **AWS region** (e.g., `eu-west-1`).
3. Expand AWS options.
4. **AWS Disk Size**: maybe 40GB to start.
5. **Instance Type**: e.g., `t2.small`.
6. **AWS Profile**: select `Devpod` (or the name you chose).
7. **AWS VPC ID**: add your VPC.
8. You can leave the rest as defaults.

Click **Add Provider**.

![added_new_provider](new_provider.png#center)

## Testing a Deployment 🧪

### Deploy

We’ll run a quick test using one of the prebuilt Docker images:

1. Go to **Workspaces** in DevPod.
2. Click **Create Workspace**.
3. Pick your new **AWS** provider.
4. Choose your preferred IDE (VS Code, etc.).
5. On the right, pick a quickstart example (e.g., Python). 🐍
6. Click **Create Workspace**.

![new_worspace](new_worskapce.png#center)

Wait a few moments, and your cloud-based environment will pop up in VS Code. 🎊

![vscode](vscode.png#center)

### Stop

When you’re not using it, click **Stop** to shut down the EC2 instance. You’ll only pay for storage—no compute time. Great for the wallet. 💰

![Stopped Instance](stopped_instance.png#center)

### Delete

Deleting the workspace removes all AWS resources for that environment, so you won’t pay a dime. But you’ll have to redeploy if you want to use it again. ♻️

![Delete Instance](delete_instance.png#center)

## Conclusion 💡

By combining **DevContainers** and **DevPod** on **AWS**, you can build flexible, self-managed dev environments that scale as you grow—without getting boxed in by vendor-specific platforms. Wave goodbye to “It works on my machine!” woes, and say hello to frictionless coding. 🚀✨

Happy building and exploring! 🎉
