# Exam 1 Prep — Question Checklist

**Round 1 tracking:** Steve answers even questions, Or answers odd questions.
**Round 2:** swap — Steve gets Or's questions, Or gets Steve's questions.

---

## Runtime Architecture

- [x] Describe a three-tier application — **Steve (Q5)**
- [x] When would you choose a monolithic architecture over microservices? — **Steve (Q11)**

## Git

- [x] How would you define a branch?
- [x] What is a pull request?
- [x] What does `--ff-only` mean? `--no-ff`? When should each be used? — **Or (Q17)**
- [x] Describe how several people should use Git to collaborate on the same project — **Steve (Q13)**
- [x] What is a conflict? When does it happen? How should it be handled? — **Or (Q3)**
- [x] What is the difference between back-merge and rebase? What would you recommend? — **Or (Q6)**
- [ ] Explain the different types of merges in Git: fast-forward, three-way merge, rebase, and back-merge
- [x] Compare trunk-based development, Git Flow, and the Mainline Model. When would you use each? — **Steve (Q15)**
- [x] Your company ships on-premises software to enterprise clients on different versions. Which branching strategy would you recommend and why? — **Or (Q21)**
- [x] How do you handle a bad commit that has been pushed to a shared branch? — **Steve (Q8)**
- [x] What is the difference between `git reset`, `git revert`, and `git checkout`? — **Or (Q1)**
- [x] Compare GitHub vs GitLab — **Steve (Q10)**
- [x] You're on a feature branch and main has been updated with important changes. What do you do? — **Or (Q19)**
- [ ] You accidentally committed sensitive data and pushed it. How do you handle this?

## Docker

- [x] What is Docker and how does it differ from virtual machines? — **Steve (Q2)**
- [x] What is the difference between a container and an image? — **Or (Q12)**
- [ ] Explain the Docker container lifecycle (build / init / run time)
- [ ] What happens when the main process inside a container stops?
- [x] How are Docker images built and what are image layers? — **Or (Q16)**
- [x] Why is it important to minimize Docker image size? — **Steve (Q22)**
- [x] How do you choose between different base images (fat vs thin)? — **Steve (Q20)**
- [ ] What is an Alpine image and when should it be used?
- [ ] If an Nginx image weighs 100MB and you deploy 3 containers, will total disk usage be 300MB?
- [x] What is a Dockerfile and what are the best practices for writing one? — **Steve (Q18)**
- [x] What is the difference between ENTRYPOINT and CMD? How can you overwrite each? — **Or (Q9)**
- [x] What is the purpose of WORKDIR? What happens if the directory doesn't exist? — **Or (Q14)**
- [ ] What is the difference between ADD and COPY?
- [ ] When should you use a volume vs COPY?
- [x] What is a multi-stage build and when do you use it? — **Or (Q7)**
- [ ] How do ARGs work in a Dockerfile?
- [x] What are Docker volumes? Explain the three types and their use cases — **Steve (Q4)**
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
