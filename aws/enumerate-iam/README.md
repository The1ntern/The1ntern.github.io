# Adjustments to `enumerate-iam`

The intention of this article is to describe how an Operator can adjust the tool [enumerate-iam](https://github.com/andresriancho/enumerate-iam) to only target specific AWS services.

Amazon Web Services (AWS) contains, as of 2025, 240 fully featured services<sup><a href="https://www.aboutamazon.com/what-we-do/amazon-web-services" target="_blank" rel="noopener noreferrer">1</a></sup>. In the event AWS cloud credentials are capturing during a capture the flag or authorized engagement the permissions for the credentials will be unknown. This is where the tool can assist. 

[enumerate-iam](https://github.com/andresriancho/enumerate-iam) will collect the current **read-only** AWS API calls available and execute a brute-force attack against each call. This will allow the Operator to determine which permissions exist. Knowing these calls can assist in finding the services to target further.

## Installation

Generally the instructions provided [here](https://github.com/andresriancho/enumerate-iam#installation) are sufficient. The commands executed below are strictly for personal preference.

```bash
git clone git@github.com:andresriancho/enumerate-iam.git
cd enumerate-iam/
python3 -m venv .
source bin/activate
pip install -r requirements.txt
```

## Execution

Follow the instructions provided in the [original GitHub](https://github.com/andresriancho/enumerate-iam?tab=readme-ov-file#updating-the-api-calls) to pull the current AWS calls. Once that is complete, the files can be modified to only target specific services.

Your current directory should be `.../enumerate-iam/enumerate-iam` with the following files:

![`ls` in `enumerate-iam` directory](images/ls_cmd.png)

[Back to Home](https://blog.the1ntern.net)