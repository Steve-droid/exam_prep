# Exam 1 Prep — Question Checklist

## Runtime Architecture
- [ ] Describe a three-tier application
- [ ] When would you choose a monolithic architecture over microservices?

## Git
- [ ] How would you define a branch?
- [ ] What is a pull request?
- [ ] What does `--ff-only` mean? `--no-ff`? When should each be used?
- [ ] Describe how several people should use Git to collaborate on the same project
- [ ] What is a conflict? When does it happen? How should it be handled?
- [ ] What is the difference between back-merge and rebase? What would you recommend?
- [ ] Explain the different types of merges in Git: fast-forward, three-way merge, rebase, and back-merge
- [ ] Compare trunk-based development, Git Flow, and the Mainline Model. When would you use each?
- [ ] Your company ships on-premises software to enterprise clients on different versions. Which branching strategy would you recommend and why?
- [ ] How do you handle a bad commit that has been pushed to a shared branch?
- [ ] What is the difference between `git reset`, `git revert`, and `git checkout`?
- [ ] Compare GitHub vs GitLab
- [ ] You're on a feature branch and main has been updated with important changes. What do you do?
- [ ] You accidentally committed sensitive data and pushed it. How do you handle this?

## Docker
- [ ] What is Docker and how does it differ from virtual machines?
- [ ] What is the difference between a container and an image?
- [ ] Explain the Docker container lifecycle (build / init / run time)
- [ ] What happens when the main process inside a container stops?
- [ ] How are Docker images built and what are image layers?
- [ ] Why is it important to minimize Docker image size?
- [ ] How do you choose between different base images (fat vs thin)?
- [ ] What is an Alpine image and when should it be used?
- [ ] If an Nginx image weighs 100MB and you deploy 3 containers, will total disk usage be 300MB?
- [ ] What is a Dockerfile and what are the best practices for writing one?
- [ ] What is the difference between ENTRYPOINT and CMD? How can you overwrite each?
- [ ] What is the purpose of WORKDIR? What happens if the directory doesn't exist?
- [ ] What is the difference between ADD and COPY?
- [ ] When should you use a volume vs COPY?
- [ ] What is a multi-stage build and when do you use it?
- [ ] How do ARGs work in a Dockerfile?
- [ ] What are Docker volumes? Explain the three types and their use cases
- [ ] How do you create a bridge network in Docker?
- [ ] How does a container obtain an internal IP address?
- [ ] How can you assign a specific IP address to a container?
- [ ] How do you link one container with another? What are the modern alternatives?
- [ ] How do you run/access/check/stop a container and remove an image?
- [ ] How can you monitor resource utilization of a container?
- [ ] How do you save changes made to a running container?
- [ ] How can you debug a container and view its processes from inside and outside?
- [ ] Why might a container exit immediately after starting?
- [ ] A container exited unexpectedly without a message. How do you troubleshoot?
- [ ] Your container keeps crashing. Walk through your troubleshooting process
- [ ] Design exercise: two containers with all three volume types, port mapping, networking, persistence
