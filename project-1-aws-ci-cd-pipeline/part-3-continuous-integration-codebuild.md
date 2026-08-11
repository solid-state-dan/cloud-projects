# Part 3: Continuous integration (CodeBuild)

## Project Overview

This is part three of a series of projects where I'm building an AWS CI/CD pipeline. Here, I used AWS CodeBuild to automate the build process in a CI/CD pipeline. "Building" basically means packaging the web app's files into a single compressed file (like a zip file) so it is ready for deployment. This project focuses entirely on setting up CodeBuild to handle that packaging automatically. I will cover the actual deployment in the next part!

### Key tools and concepts

Services I used include CodeBuild, CodeConnection, CodeArtifact, S3, EC2 and GitHub. Key concepts I learnt include the build process, buildspec.yml and using CodeConnection to connect AWS to GitHub.

### Project reflection

This project took me about 3 hours to complete. Getting the build process to run successfully was definitely the trickiest part, but it was incredibly rewarding to finally see it work.

## Project Walkthrough

{% stepper %}
{% step %}
## CodeBuild Setup

AWS CodeBuild is a continuous integration (CI) service that compiles and packages code. Essentially, CodeBuild takes the raw web app code and automatically packages it so computers can run it. Automating this saves engineers from doing it manually before every deploy.

For this setup, I pointed CodeBuild directly to the GitHub repository where the web app code is stored.

### Connecting CodeBuild with GitHub

While AWS offers a few ways to connect to GitHub—like personal access tokens or OAuth apps—I chose to use a GitHub App. It is the simplest and most secure option because AWS manages all the authentication tokens in the background. Setup was straightforward: I just logged into GitHub once through the AWS-managed application interface.

### Repository Access via CodeConnections

The actual connection between AWS and GitHub was handled by AWS CodeConnections. It is a handy service for linking AWS to third-party tools. By setting it up, my CodeBuild project gained the exact permissions it needed to access my GitHub repository without exposing sensitive credentials.
{% endstep %}

{% step %}
## CodeBuild Configurations

### Environment

The Environment configuration sets up the temporary EC2 instance that powers the build process. It covers key parameters like the provisioning model, compute capacity, operating system, base image, and service role. These settings dictate exactly how the instance boots up and executes the build tasks. A critical step here is ensuring the assigned service role has the exact IAM permissions the instance needs to interact with other AWS services.

### Artifacts & Storage

Build artifacts are the files or resources generated during the CodeBuild process. They are a crucial piece of the puzzle because they are needed later in the CI/CD pipeline. In this case, the build process outputs a compressed file of the web app, which I will use for deployment in the next stage. To store and manage these outputs securely, I configured an Amazon S3 bucket as the artifact repository.

### Artifact Packaging

During the setup, I opted to compress the output artifacts into a Zip file. This approach keeps all the application files neatly organized into a single archive, which makes package management much simpler. Additionally, zipping the files compresses the web app's overall size, making transfers faster and storage more efficient.

### Monitoring & Logging

For monitoring, I enabled Amazon CloudWatch Logs to capture the commands executed during the build along with any errors that might occur. Having this running is incredibly helpful for troubleshooting and pinpointing exactly where a build might fail.
{% endstep %}

{% step %}
## Defining the Build Instructions

### buildspec.yml Configuration

My first build actually failed because CodeBuild couldn't find a buildspec.yml file in the root directory of my source code. This file is essential because it acts as the instruction manual, telling CodeBuild exactly how to execute the build process step by step.

### Build Phases

* Phases 1 & 2 (Setup): Installs Java and authenticates with CodeArtifact for secure dependency access.
* Phase 3 (Compile): Compiles the web app's source code.
* Phase 4 (Package): Bundles the compiled application into a single compressed artifact.
{% endstep %}

{% step %}
## Troubleshooting & Success

I hit one more quick roadblock on my second try. The build failed while trying to compile the app using the command `mvn -s settings.xml compile`. It turns out CodeBuild didn't have the right permissions to talk to CodeArtifact yet. I fixed the issue by updating my CodeBuild service role with a policy allowing CodeArtifact access. This privilege gave CodeBuild the exact permissions it required to interact with the repository. So, I ran it again, and the build finally succeeded!

### Verifying the Build

To make sure everything ran properly, I took a look inside my S3 artifacts bucket. Seeing the zipped package there confirmed that the CodeBuild project was a success. The process had successfully fetched the source code from GitHub, compiled it, compressed it, and saved the final artifact to S3.
{% endstep %}
{% endstepper %}
