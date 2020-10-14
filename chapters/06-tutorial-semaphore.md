
\newpage

## 4.4 Implementing a CI/CD Pipeline With Semaphore

In this section, we’ll learn about Semaphore and how to use it to build cloud-based CI/CD pipelines.

### 4.4.1 Introduction to Semaphore

For a long time, developers looking for a CI/CD tool had to choose between power and ease of use.

On one hand, there was Jenkins which can do just about anything, but is difficult to use and requires dedicated ops teams to configure, maintain and scale.

On the other hand, there were several hosted services that let developers just push their code and not worry about the rest of the process. However, these services are usually limited to running simple build and test steps, and would often fall short in need of more elaborate continuous delivery workflows, which is often the case with containers.

Semaphore (_[https://semaphoreci.com](https://semaphoreci.com/?utm_source=ebook&utm_medium=pdf&utm_campaign=cicd-docker-kubernetes-semaphore)_) started as one of the simple hosted CI services, but eventually evolved to support custom continuous delivery pipelines with containers, while retaining a way of being easy to use by any developer, not just dedicated ops teams. As such, it removes all technical barriers to adopting continuous delivery at scale:

- It's a cloud-based service that scales on demand. There's no software for you to install and maintain.
- It provides a visual interface to model custom CI/CD workflows quickly.
- It's the fastest CI/CD service, due to being based on dedicated hardware instead of common cloud computing services.
- It's free for open source and small private projects.

The key benefit of using Semaphore is increased team productivity. Since there is no need to hire supporting staff or expensive infrastructure, and it runs CI/CD workflows faster than any other solution, companies that adopt Semaphore report a very large, 41x ROI comparing to their previous solution [^roi].

We'll learn about Semaphore's features as we go hands-on in this chapter.

[^roi]: Whitepaper: The 41:1 ROI of Moving CI/CD to Semaphore (_[https://semaphoreci.com/resources/roi](https://semaphoreci.com/resources/roi?utm_source=ebook&utm_medium=pdf&utm_campaign=cicd-docker-kubernetes-semaphore)_)

### 4.4.1 Creating a Semaphore Account

To get started with Semaphore:

- Go to _[https://semaphoreci.com](https://semaphoreci.com?utm_source=ebook&utm_medium=pdf&utm_campaign=cicd-docker-kubernetes-semaphore)_ and click to sign up with your GitHub account.
- GitHub will ask you to let Semaphore access your profile information. Allow this so that Semaphore can create an account for you.
- Semaphore will walk you through the process of creating an organization. Since software development is a team sport, all Semaphore projects belong to an organization. Your organization will have its own domain, for example, `awesomecode.semaphoreci.com`.
- Semaphore will ask you to choose between a time-limited free trial with unlimited capacity, a free plan, and an open-source plan. This chapter will demonstrate a workflow using Semaphore's Docker Registry, which is available within a free trial or a paid account. However you can easily replace it with a free public registry like Docker Hub and implement the same workflow with an open source account.
- Finally, you'll be greeted with a quick product tour.

### 4.4.2 Creating a Semaphore Project For The Demo Repository

We assume that you have previously forked the demo project from _[https://github.com/semaphoreci-demos/semaphore-demo-cicd-kubernetes](https://github.com/semaphoreci-demos/semaphore-demo-cicd-kubernetes)_ to your GitHub account.

On Semaphore, follow the prompts to create a project. The first time you do this, you will see a screen which asks you to choose between connecting Semaphore to either your public, or both public and private repositories on GitHub:

![Authorizing Semaphore to access your GitHub repositories](./figures/05-github-repo-auth.png){ width=95% }

To keep things simple, select the "Public repositories" option. If you later decide that you want to use Semaphore with your private projects as well, you can extend the permission at any time.

Next, Semaphore will present you a list of repositories to choose from as the source of your project:

![Choosing a repository to set up CI/CD for](./figures/05-choose-repo.png){ width=95% }

In the search field, start typing `semaphore-demo-cicd-kubernetes` and choose that repository.

Semaphore will quickly initialize the project. Behind the scenes, it will set up everything that's needed to know about every Git push automatically pull the latest code — without you configuring anything.

The next screen lets you invite collaborators to your project. Semaphore mirrors access permissions of GitHub, so if you add some people to the GitHub repository later, you can "sync" them inside project settings on Semaphore.

![Add collaborators](./figures/05-sem-add-collaborators.png){ width=95% }

Click on *Go to Workflow Builder*. Semaphore will ask you if you want to use the existing pipelines or create one from scratch. At this point, you can choose to use the existing configuration to get directly to the final workflow. In this chapter, however, we want to learn how to create the pipelines so we’ll make a fresh start.

![Start from scratch or use existing pipeline](./figures/05-sem-existing-pipeline.png){ width=95% }

Click on the option to configure the project from scratch.

### 4.4.3 The Semaphore Workflow Builder

To make the process of creating projects easier, Semaphore provides starter workflows for popular frameworks and languages. Choose the "Build Docker" workflow and click on *Run this workflow*.

![Choosing a starter workflow](./figures/05-sem-starter-workflow.png){ width=95% }

Semaphore will immediately start the workflow. Wait a few seconds and your first Docker image is ready, congratulations!

![Starter run](./figures/05-sem-starter-run.png){ width=95% }

Since we haven’t told Semaphore where to store the image yet, it’s lost as soon as the job ends. We’ll correct that next.

See the *Edit Workflow* button on the top right corner? Click it to open the Workflow Builder.

Now it’s a good moment to learn the basic concepts of Semaphore by exploring the Workflow Builder.

![Workflow Builder overview](./figures/05-sem-wb-overview.png){ width=95% }

**Pipelines**

Pipelines are represented in Workflow Builder as big gray boxes. Pipelines organize the workflow in blocks that are executed from left to right. Each pipeline usually has a specific objective such as test, build, or deploy. Pipelines can be chained together to make complex workflows.

**Agent**

The agent is the combination of hardware and software that powers the pipeline. The *machine type* determines the amount of CPUs and memory allocated to the virtual machine[^vm-types]. The operating system is controlled by the *Environment Type* and *OS Image* settings.

The default machine is called `e1-standard-2` and has 2 CPUs, 4 GB RAM, and runs a custom Ubuntu 18.04 image.

[^vm-types]: To see all the available machines, go to [https://docs.semaphoreci.com/ci-cd-environment/machine-types](https://docs.semaphoreci.com/ci-cd-environment/machine-types/?utm_source=ebook&utm_medium=pdf&utm_campaign=cicd-docker-kubernetes-semaphore)

**Jobs and Blocks**

Blocks and jobs define what to do at each step. Jobs define the commands that do the work. Blocks contain jobs with a common objective and shared settings.

Jobs inherit their configuration from their parent block. All the jobs in a block run in parallel, each in its isolated environment. If any of the jobs fails, the pipeline stops with an error.

Blocks run sequentially, once all the jobs in the block complete, the next block starts.

### 4.4.4 The Continous Integration Pipeline

We talked about the benefits of CI/CD in chapter 3. In the previous section, we created our very first pipeline. In this section, we’ll extend it with tests and a place to store the images.

At this point, you should be seeing the Workflow Builder with the Docker Build starter workflow. Click on the *Build* block so we can see how it works.

![Build block](./figures/05-sem-build-block.png){ width=95% }

Each line on the job is a command to execute. The first command in the job is `checkout`, which is a built-in script that clones the repository at the correct revision[^sem-toolbox]. The next command, `docker build`, builds the image using our `Dockerfile`.

[^sem-toolbox]: You can find the complete Semaphore toolbox at [https://docs.semaphoreci.com/reference/toolbox-reference](https://docs.semaphoreci.com/reference/toolbox-reference/?utm_source=ebook&utm_medium=pdf&utm_campaign=cicd-docker-kubernetes-semaphore)

**Note**: Long commands have been broken down into two or more lines with backslash (\\) to fit on the page. Semaphore expects one command per line, so when typing them, remove the backslashes and newlines.

Replace the contents of the job with the following commands:

```bash
checkout

docker login \
  -u $SEMAPHORE_REGISTRY_USERNAME \
  -p $SEMAPHORE_REGISTRY_PASSWORD \
  $SEMAPHORE_REGISTRY_URL

docker pull \
  $SEMAPHORE_REGISTRY_URL/demo:latest || true

docker build \
  --cache-from $SEMAPHORE_REGISTRY_URL/demo:latest \
  -t $SEMAPHORE_REGISTRY_URL/demo:$SEMAPHORE_WORKFLOW_ID .

docker push \
  $SEMAPHORE_REGISTRY_URL/demo:$SEMAPHORE_WORKFLOW_ID
```

Each command has its purpose:

1. Clones the repository with `checkout`.
2. Logs in the Semaphore private Docker registry[^docker-registry].
3. Pulls the Docker image tagged as `latest`.
4. Builds a newer version of the image using the latest code.
5. Pushes the new image to the registry.

The perceptive reader will note that we introduced special environment variables; these come predefined in every job[^environment]. The variables starting with `SEMAPHORE_REGISTRY_*` are used to access the private registry. Also, we’re using `SEMAPHORE_WORKFLOW_ID`, which is guaranteed to be unique for each run, to tag the image.

[^docker-registry]: Semaphore's built-in Docker registry is available under a paid plan or free trial. If you're using a free or open source plan, use an external service like Docker Hub instead. The pipeline will be slower but the workflow will be the same.
[^environment]: The full environment reference can be found at [https://docs.semaphoreci.com/ci-cd-environment/environment-variables](https://docs.semaphoreci.com/ci-cd-environment/environment-variables/?utm_source=ebook&utm_medium=pdf&utm_campaign=cicd-docker-kubernetes-semaphore)

![Build block](./figures/05-sem-build-block-2.png){ width=95% }

Now that we have a Docker image that we can test let’s add a second block. Click on the *+Add Block* dotted box.

The Test block will have jobs:

- Static tests.
- Integration tests.
- Functional tests.

The general sequence is the same for all tests:

1. Pull the image from the registry.
2. Start the container.
3. Run the tests.

Blocks can have a *prologue* in which we can place shared initialization commands. Open the prologue section on the right side of the block and type the following commands, which will be executed before each job:

``` bash
docker login \
  -u $SEMAPHORE_REGISTRY_USERNAME \
  -p $SEMAPHORE_REGISTRY_PASSWORD \
  $SEMAPHORE_REGISTRY_URL

docker pull \
  $SEMAPHORE_REGISTRY_URL/demo:$SEMAPHORE_WORKFLOW_ID
```

Next, rename the first job as “Unit test” and type the following command, which runs JSHint, a static code analysis tool:

``` bash
docker run -it \
  $SEMAPHORE_REGISTRY_URL/demo:$SEMAPHORE_WORKFLOW_ID \
  npm run lint
```

Next, click on the *+Add another job* link below the job to create a new one called “Functional test”. Type these commands:

``` bash
sem-service start postgres

docker run --net=host -it \
  $SEMAPHORE_REGISTRY_URL/demo:$SEMAPHORE_WORKFLOW_ID \
  npm run ping

docker run --net=host -it \
  $SEMAPHORE_REGISTRY_URL/demo:$SEMAPHORE_WORKFLOW_ID \
  npm run migrate
```

This job tests two things: that the container connects to the database (`ping`) and that it can create the tables (`migrate`). Obviously, we’ll need a database for this to work; fortunately, we have `sem-service`, which lets us start database engines like MySQL, Postgres, or MongoDB with a single command[^sem-service].

[^sem-service]: For the complete list of services sem-service can manage check: [https://docs.semaphoreci.com/ci-cd-environment/sem-service-managing-databases-and-services-on-linux/](https://docs.semaphoreci.com/ci-cd-environment/sem-service-managing-databases-and-services-on-linux/?utm_source=ebook&utm_medium=pdf&utm_campaign=cicd-docker-kubernetes-semaphore)

Finally, add a third job called “Integration test” and type these commands:

``` bash
sem-service start postgres

docker run --net=host -it \
  $SEMAPHORE_REGISTRY_URL/demo:$SEMAPHORE_WORKFLOW_ID \
  npm run test
```

This last test runs the code in `src/database.test.js`, which checks if the application can write and delete rows in the database.

![Test block](./figures/05-sem-test-block.png){ width=95% }

Create the third block in the pipeline and call it “Push”. This last job will tag the current Docker image as `latest`. Type these commands in the job:

``` bash
docker login \
  -u $SEMAPHORE_REGISTRY_USERNAME \
  -p $SEMAPHORE_REGISTRY_PASSWORD $SEMAPHORE_REGISTRY_URL

docker pull \
  $SEMAPHORE_REGISTRY_URL/demo:$SEMAPHORE_WORKFLOW_ID

docker tag \
  $SEMAPHORE_REGISTRY_URL/demo:$SEMAPHORE_WORKFLOW_ID \
  $SEMAPHORE_REGISTRY_URL/demo:latest

docker push \
  $SEMAPHORE_REGISTRY_URL/demo:latest
```

![Push block](./figures/05-sem-push-block.png){ width=95% }

This completes the setup of the CI pipeline.

### 4.4.5 Your First Build

We’ve covered a lot of things in a few pages; here, we have the chance to pause for a little bit and try the CI pipeline. Click on the *Run the workflow* button on the top-right corner and then click on *Start*.

![Run this workflow](./figures/05-sem-run-workflow.png){ width=95% }

![CI pipeline done](./figures/05-sem-ci-pipeline.png){ width=95% }

Wait until the pipeline is complete then go to the top level of the project. Click on the *Docker Registry* button and open the repository to verify that the Docker image is there.

![Docker registry](./figures/05-sem-registry.png){ width=95% }
