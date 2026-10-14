# AWS Elastic Container Service

## What?

- Amazon Elastic Container Service (ECS) is a managed service for running and coordinating Docker containers.
- Think of ECS as a building manager: it decides where containers run, keeps the requested number running, and replaces unhealthy ones.
- An **ECS cluster** is a logical group of compute capacity where containers can run.
- A **task definition** is the container blueprint: it describes the image, CPU, memory, ports, environment variables, and permissions.
- A **task** is a running copy of a task definition.
- An **ECS service** keeps a chosen number of task copies running and can replace failed tasks.

## How?

- Build a Docker image and store it in a registry such as Amazon ECR.
- Create an ECS cluster and choose how containers will run:
	- **Fargate:** AWS manages the servers; you pay for the resources used by the tasks.
	- **EC2:** You manage the EC2 instances that provide the container capacity.
- Create a task definition that points to the container image and defines how the container should run.
- Create an ECS service from the task definition.
- Choose a VPC, subnets, security groups, and an optional Application Load Balancer.
- ECS schedules tasks, monitors their health, and maintains the desired number of replicas.

## Why?

- ECS removes the need to start and manage containers manually on individual servers.
- Services can scale the number of container replicas up or down as demand changes.
- Failed or unhealthy tasks can be replaced automatically.
- ECS integrates with ECR, IAM, CloudWatch, load balancers, and Auto Scaling.
- Fargate is useful when you want to run containers without managing EC2 instances.

> **Analogy:** The task definition is a recipe, a task is a prepared meal, and an ECS service makes sure the kitchen always has the requested number of meals ready.

> **Typical flow:** Docker image -> ECR -> ECS task definition -> ECS service -> running containers

## ECS vs. App Runner

- **App Runner** is a simpler application deployment service: provide a source repository or container image, and AWS handles most of the infrastructure.
- **ECS** is a container orchestration service: you define task definitions, services, networking, scaling, and deployment behavior in more detail.
- **Control:** App Runner offers fewer configuration options; ECS provides finer control over tasks, networking, load balancers, IAM roles, and deployment strategies.
- **Infrastructure:** App Runner hides most infrastructure management; ECS can run on Fargate for serverless containers or on EC2 for more control over the hosts.
- **Networking:** App Runner needs a VPC connector for outbound access to private resources; ECS tasks can be placed directly inside VPC subnets.
- **Scaling:** Both can scale containers, but ECS supports more advanced service discovery, placement, and scaling patterns.
- **Best for App Runner:** Simple web applications and APIs where fast deployment and low operational effort matter most.
- **Best for ECS:** Multi-container applications, complex networking, custom deployment strategies, and workloads that need detailed orchestration control.

> **Analogy:** App Runner is like ordering a managed food truck: you provide the recipe and AWS handles the kitchen. ECS is like managing a full restaurant: you have more responsibility, but also more control over how everything operates.

| Feature | App Runner | ECS |
| --- | --- | --- |
| Setup effort | Lower | Higher |
| Infrastructure control | Limited | Detailed |
| Container orchestration | Simplified | Advanced |
| Compute choices | Fully managed | Fargate or EC2 |
| Best fit | Simple web apps and APIs | Complex or highly customized container workloads |

