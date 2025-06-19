# The Great Git Glitch: How I Conquered a "Large File" Error While Learning Terraform

## 🗓️ Date Encountered: June 20, 2025
## 🏷️ Category: Terraform, Git

---

## 🧐 The Problem I Faced

As I'm diving deeper into Infrastructure as Code (IaC) with Terraform and AWS, I've been diligently working on my configurations. Naturally, I want to keep my code version-controlled on GitHub. I meticulously set up my `.gitignore` file, added my Terraform files, committed them, and felt ready to push.

Then, Git threw a curveball I wasn't expecting. After hitting `git push origin main`, my terminal screamed:


As I'm diving deeper into Infrastructure as Code (IaC) with Terraform and AWS, I've been diligently working on my configurations. Naturally, I want to keep my code version-controlled on GitHub. I meticulously set up my `.gitignore` file, added my Terraform files, committed them, and felt ready to push.

Then, Git threw a curveball I wasn't expecting. After hitting `git push origin main`, my terminal screamed:

remote: error: File ec2/windows_amd64/terraform-provider-aws_v5.98.0_x5.exe is 681.12 MB; this exceeds GitHub's file size limit of 100.00 MB
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
To https://github.com/Nallagachu/terraform.git
! [remote rejected] main -> main (pre-receive hook declined)
error: failed to push some refs to 'https://github.com/Nallagachu/terraform.git'


"Wait, what?!" I thought. "But I have a `.gitignore` file! Why is Git complaining about a file I'm explicitly telling it to ignore?" This seemingly simple problem led me down a rabbit hole of Git history, environment variables, and crucial learning moments. If you've ever faced a similar head-scratcher while learning, this detailed guide is for you!

---

### Understanding the Culprit: The Uninvited Terraform Provider Executable

The error message was crystal clear: `ec2/windows_amd64/terraform-provider-aws_v5.98.0_x5.exe` was too big.

**What is this behemoth?**
When you execute `terraform init` in your project, Terraform intelligently downloads the necessary **provider plugins** for the services you're interacting with (in my case, AWS). These providers are binary executables (like `.exe` files on Windows) that allow Terraform to communicate with the cloud provider's API. This specific file was the AWS provider for Windows on an AMD64 architecture.

**The Fatal Flaw: Committed Before Ignored**
Here's the critical piece of the puzzle: While my `.gitignore` file was ready to prevent *future* tracking of such files, this particular provider executable had been downloaded and then, at some point, was **staged and committed into my Git repository's history** *before* the `.gitignore` rule took effect for it.

**Understanding Git's History:**
This is where many learners (myself included!) get tripped up. Git doesn't just care about the current state of your files; it meticulously tracks every single version in its commit history. Even if you `rm` (delete) a file from your local working directory, its presence in a *past commit* means it's still part of your repository's "total size" when you try to push that history to GitHub. My repository was essentially carrying a huge, invisible backpack of old data.

---

### My Initial (Failing) Instincts

My first, logical reaction was to simply delete the file from my project folder:

```bash
LENOVO@pandujyo MINGW64 /c/devops/repos/terraform (main)
$ rm ec2/windows_amd64/terraform-provider-aws_v5.98.0_x5.exe

LENOVO@pandujyo MINGW64 /c/devops/repos/terraform (main)
$ git status
On branch main
nothing to commit, working tree clean
"Perfect!" I thought. git status showed a clean slate. With renewed hope, I tried to push again:

Bash

LENOVO@pandujyo MINGW64 /c/devops/repos/terraform (main)
$ git add . ; git commit -m "pushing code to git " ; git push origin main
# ... (Boom! Same GitHub error. My heart sank.)
This confirmed my dawning realization: deleting the file from my current working directory was like cleaning the top layer of a dusty old photo album. The dusty old photos (the large file in past commits) were still there, making the whole album too heavy to mail. I needed to fundamentally rewrite the album's history.

The Powerful Fix: Rewriting Git History with git-filter-repo
To truly purge the large file, I needed a tool that could surgically remove it from every single commit where it existed. The modern, highly recommended tool for this task is git-filter-repo. It's a Python script that's significantly faster and generally safer than its predecessor, git filter-branch.

🚨 A CRITICAL WARNING About Rewriting History!

ALWAYS BACK UP YOUR REPOSITORY FIRST! Before touching your Git history, make a complete copy of your project folder. This is your safety net.
TEAM ENVIRONMENTS: DO NOT DO THIS LIGHTLY! If your repository is shared with a team, you absolutely MUST coordinate with them before rewriting history. This operation changes commit IDs, and without coordination, it will cause significant headaches and potential data loss for collaborators. For a personal learning repository, it's generally safe to proceed after backing up.
Here's the detailed, step-by-step process I followed to fix my repository:

Step 1: The Unexpected Detour - Getting Python & Pip Working in Git Bash
Before I could even use git-filter-repo, I hit another roadblock: my Git Bash terminal couldn't find python or pip, even though they worked fine in PowerShell. This often happens because Git Bash uses its own environment configuration, and your Python installation might not be in its PATH.

The Initial Error:

Bash

LENOVO@pandujyo MINGW64 /c/devops/repos/terraform (main)
$ pip install git-filter-repo
bash: pip: command not found
Finding Python's Home:

Open PowerShell (not Git Bash) and run:

PowerShell

python -c "import sys; print(sys.executable)"
This showed me my Python executable was at:
C:\Users\LENOVO\AppData\Local\Programs\Python\Python313\python.exe

From this, I deduced my Python installation directory (C:\Users\LENOVO\AppData\Local\Programs\Python\Python313) and the Scripts directory where pip.exe resides (C:\Users\LENOVO\AppData\Local\Programs\Python\Python313\Scripts).

Integrating Python into Git Bash's PATH:

Open your Git Bash terminal.

Create or edit your ~/.bashrc file. This file runs commands when Git Bash starts.

Bash

nano ~/.bashrc
(If nano isn't installed, you can open C:\Users\LENOVO\.bashrc directly in VS Code or Notepad).

Add the Python paths. Paste these lines into the file. Crucially, replace the paths with your actual Python installation directories, and remember to convert C:\ to /c/ and use forward slashes!

Bash

# Add Python to PATH for Git Bash
export PATH="/c/Users/LENOVO/AppData/Local/Programs/Python/Python313:$PATH"
export PATH="/c/Users/LENOVO/AppData/Local/Programs/Python/Python313/Scripts:$PATH"
Save and Exit nano: (Ctrl + O, Enter, Ctrl + X).

Restart Git Bash: The easiest way to apply these changes is to close your current Git Bash window and open a new one.

Verify Python and Pip are working in Git Bash:
In your new Git Bash terminal, test it out:

Bash

python --version
pip --version
Finally, they both showed their versions, confirming the environment was ready.

Bash

LENOVO@pandujyo MINGW64 /c/devops/terraform
$ python --version
pip --version
Python 3.13.5
pip 25.1.1 from C:\Users\LENOVO\AppData\Local\Programs\Python\Python313\Lib\site-packages\pip (python 3.13)
Step 2: Install git-filter-repo
With pip now accessible, installing the necessary tool was a breeze:

Bash

pip install git-filter-repo
Step 3: Execute git-filter-repo to Purge History
Now for the main event! First, ensure you're in the root directory of your Git repository (the one that had the large file, which was /c/devops/repos/terraform):

Bash

cd /c/devops/repos/terraform # Confirm you're in your project folder
Then, run the git-filter-repo command to rewrite your history:

Bash

git filter-repo --path ec2/windows_amd64/terraform-provider-aws_v5.98.0_x5.exe --invert-paths --force
--path <file_path>: Targets the specific file you want to remove.
--invert-paths: Crucially tells the command to keep everything EXCEPT this path, effectively deleting it from history.
--force: Required because we're performing a deep history rewrite.
You'll see messages indicating that Git objects are being rewritten. This process might take a few moments.

Step 4: Clean Up Git's Internal References
Even after rewriting history, Git holds onto old, "unreachable" objects for a while in its reflog. To fully reclaim disk space and ensure the old, large file is truly gone, you need to clean up:

Bash

git reflog expire --expire=now --all
git gc --prune=now
git reflog expire --expire=now --all: Immediately clears out your reflog (which logs all local changes to your branches).
git gc --prune=now: Runs Git's garbage collector, which deletes unreachable objects (like the data from the large file) and optimizes the repository.
Step 5: The Mystery of the Missing Remote - Troubleshooting fatal: 'origin' does not appear...
After the history rewrite, I encountered one last hurdle when trying to push:

Bash

fatal: 'origin' does not appear to be a git repository
fatal: Could not read from remote repository.
This meant my local Git repository had lost its reference to origin (the name for my remote GitHub repository). Sometimes, deep history rewrites can disrupt these connections.

How to Fix:

Check if origin exists:

Bash

git remote -v
This command confirmed that origin was indeed missing from my remotes list.

Re-add the origin remote:
You'll need the HTTPS URL of your GitHub repository. Go to your repo on GitHub, click the green "Code" button, and copy the HTTPS URL (e.g., https://github.com/Nallagachu/terraform.git).

Then, add it back to your local repository:

Bash

git remote add origin [https://github.com/Nallagachu/terraform.git](https://github.com/Nallagachu/terraform.git)
(Remember to use your actual repo URL if different).

Step 6: The Final, Triumphant Push!
With the history cleaned and the origin remote re-established, it was time for the moment of truth.

🚨 FINAL WARNING: You are about to force push!
This command will overwrite the history on your main branch on GitHub. Since this was a personal learning project, it was safe. In a team setting, this would require explicit coordination.

Bash

git push origin main --force
And finally, sweet success!

Bash

LENOVO@pandujyo MINGW64 /c/devops/repos/terraform (main)
$ git push origin main --force
Enumerating objects: 51, done.
Counting objects: 100% (51/51), done.
Delta compression using up to 12 threads
Compressing objects: 100% (31/31), done.
Writing objects: 100% (51/51), 13.52 KiB | 6.76 MiB/s, done.
Total 51 (delta 17), reused 51 (delta 17), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (17/17), done.
To [https://github.com/Nallagachu/terraform.git](https://github.com/Nallagachu/terraform.git)
 * [new branch]      main -> main
My main branch on GitHub was now lean, clean, and finally free of that massive provider executable!

Key Takeaways from a Git Learning Journey
This entire experience was a masterclass in Git and environment management. Here are my biggest takeaways:

.gitignore's True Power (and Limitation): It's for preventing untracked files for future commits. It doesn't rewrite history.
Terraform Providers Are Big: Be mindful that terraform init downloads executables. Always ensure your .gitignore is set up correctly (e.g., ignoring .terraform/ and *.tfplan) before your first commit if you plan to push the repository.
git-filter-repo is Your Friend: For cleaning history, this is the tool to use. But understand its power and the need for backups.
Environment Setup Matters: Discrepancies in PATH between different terminals (like PowerShell and Git Bash) can lead to frustrating "command not found" errors. Knowing how to troubleshoot and configure your shell environment is crucial.
Git's Robustness (and Complexity): Git is incredibly powerful, but its depth means you'll hit unexpected scenarios. Persistence and understanding the underlying concepts (like history and remotes) are key.
Force Pushing is Serious: It's a tool to be used with caution, primarily in personal repos or with full team coordination.
Every error is a learning opportunity. This particular "Git Glitch" definitely leveled up my understanding of version control. I hope this detailed account helps you navigate similar challenges in your own learning journey!
