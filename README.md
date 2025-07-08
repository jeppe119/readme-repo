How to Set Up a GitHub Page for Sharing Work
This guide explains how to create a GitHub Page to share projects, documentation, or reports with your team.

Steps to Create a GitHub Page
1. Create a GitHub Account (if needed)
Sign up at github.com.

2. Create a New Repository
Click + → New repository.

Name it:

Personal/Team Site: <username>.github.io (e.g., johnsmith.github.io)

Project Page: Any name (e.g., my-project)

3. Add Your Files
You can use:

HTML/CSS/JS files (for a custom website).

Markdown (.md) (auto-converted to HTML by GitHub).

Option A: Upload Manually
Clone the repo:

bash
git clone https://github.com/<username>/<repo-name>.git
Add files (e.g., index.html, README.md, or /docs).

Push changes:

bash
git add .
git commit -m "Initial commit"
git push
Option B: Use Jekyll (for Markdown Docs)
Store files in /docs or root with a _config.yml.

GitHub renders Markdown automatically.

4. Enable GitHub Pages
Go to Repository → Settings → Pages.

Under Source, select:

main/master branch (for root files).

docs folder (if using /docs).

Click Save.

5. Access Your Page
Personal Site: https://<username>.github.io

Project Page: https://<username>.github.io/<repo-name>
(May take a few minutes to deploy.)

6. Share with Coworkers
Send them the URL.

For private repos, add them as Collaborators (Settings → Collaborators).

Tips for Better Collaboration
✔ Use Markdown (easy formatting for README.md).
✔ Add a Table of Contents for navigation.
✔ Enable GitHub Discussions (for Q&A).
✔ Use GitHub Actions for automation.
