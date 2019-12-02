# EKS CI WITH JENKINS OPERATOR

This repo contains a reference on provisioning a jenkins master with the jenkins operator.  

**Background:**

This is **_Part 2_** of a cluster setup 
see the bellow repos for next and previous steps:

* [Part 1 setting up EKS ](https://github.com/bbdsoftware/eks-bootstrap)
* [Part 2 setting up CI](https://github.com/bbdsoftware/eks-jenkins-ci)
* [Part 3 setting up CD ](https://github.com/bbdsoftware/eks-argo-cd)


*NOTE*  
PLease ensure that you have deployed the jenkins operator see 
https://github.com/bbdsoftware/eks-bootstrap#ci  
  
## Creating a master 
 [jenkins-master.yaml ](./jenkins/jenkins-master.yaml) contains the manifest for bootstarping a jenkins master using the
 jenkins operator. The manifest contains various plugins and a seed job configuration.  
 The seed job is configured to look at this git repository and load job dsl from the  jobs folder eg [spring-boot-k8s-jenkinsop-example.jenkins ](jobs/springbootk8sjenkinsopexample.jenkins)

```
....
      seedJobs:
      - id: jenkins-operator
        targets: "jobs/*.jenkins"
        description: "Jenkins Operator repository"
        credentialType: usernamePassword
        credentialID: jenkins-github
        repositoryBranch: master
        repositoryUrl: https://github.com/bbdsoftware/eks-jenkins-ci
....
```
Run

```kubectl  apply -f -n jenkins-system  -f jenkins/jenkins-master.yaml```

**References :**
* https://jenkinsci.github.io/kubernetes-operator/
* https://github.com/jenkinsci/kubernetes-operator

## Jobs via git 
Once up and running a jenkins master will be available and the seed job run.
The seed job will configure a multipipline job as defined in [spring-boot-k8s-jenkinsop-example.jenkins ](jobs/springbootk8sjenkinsopexample.jenkins)

```$xslt
       .....     
            id('spring-boot-k8s-jenkinsop-example') // IMPORTANT: use a constant and unique identifier
            url('https://github.com/bbdsoftware/eks-pring-boot-jenkinsop-example.git')
            credentials('jenkins-github')
            includes('*')
        .....
```

To add more jobs new files can be added or current ones extended. Please see job dsl https://jenkinsci.github.io/job-dsl-plugin/ for details
Once the files are pushed to git and master , the seed job will run and configure the jobs in jenkins.  
This allows jobs to eb configured as code
and the source of the truth remaining in git. This allow the jenkins master to be ephemeral

**References:**
* https://jenkinsci.github.io/job-dsl-plugin/ 
* https://github.com/jenkinsci/job-dsl-plugin

