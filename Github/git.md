## Git identity
Check Git identity locally :
```
git config user.name
git config user.email
```

Check Git identity globally
```
git config --global user.name
git config --global user.email
```

Set Git identity locally:
```
git config user.name "algofol"
git config user.email "your-email@example.com"
```

Set Git identity globally:
```
git config --global user.name "algofol"
git config --global user.email "your-email@example.com"
```

## Personal Access Tokens
Create a new one to commit and push changes from local to git repo:
1. Go to your profile -> Settings -> Developer Settings -> Fine-grained tokens
2. Create a new one with the following data:\
&nbsp;&nbsp;&nbsp;&nbsp;**Expiration**: Recommended 30 days or 1 year at most(custom)\
&nbsp;&nbsp;&nbsp;&nbsp;**Repository Access**: Only Selected Repositories\
&nbsp;&nbsp;&nbsp;&nbsp;**Repository Permissions**:\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Contents(Repository contents, commits, branches, downloads, releases, and merges)*: Read and write\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Metadata(Search repositories, list collaborators, and access repository metadata)*: Read-only\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Pull requests(Pull requests and related comments, assignees, labels, milestones, and merges)*: Read and Write
