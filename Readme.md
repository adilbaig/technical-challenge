# Solution
I've focused on solving:

1. We would like to deploy this service to a cloud provider, and automate the deployment.
2. We would like to introduce some self-healing capabilities, e.g. restart after a crash.

There's a number of ways to do this. It differs based on what cloud providers you use and what CI/CD tool you have. The two most common methods are to build images using orchestrators (like Ansible) on the cloud or build Docker images. In this solution I chose to build self-contained images using [Packer](https://packer.io). The advantage over docker is:
 1. Packer can produce both cloud-provider images and docker files. It is cloud-provider agnostic too, (unlike say AWS CloudFormation).
 2. The Docker orchestration story has changed over time (Swarm yesterday, Kubernetes today) where AWS rollouts are quite tried and true.
 3. Another advantage of using a native image is it can run multiple processes. So, for example, source code, metrics collectors and local crons can be deployed together.

For self-healing I chose to provide a [systemd service file](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-managing_services_with_systemd-unit_files). Systemd files can be used to orchestrate startup when basic dependencies are available, and have a fine-grained reboot and give-up policy.

## MakeFile
I modified the Makefile to use a Python virtualenv. That's just basic hygiene.

## A note on the other requirements

- We would like a solution that is stable, secure and maintainable.
This was quite broad. I assume for a product to be "production ready" it should include basic style checks, unit/integration tests and documentation. In the essence of time, I skipped this. 

- We would like to have monitoring that helps to identify issues.
Monitoring can be integrated using 3 points. 

1. Emit statsd metrics to collect samples of application events over time. These provide the raw data for dashboarding and can be fodder for alerts. 
2. Have a private url, like /stats that shows the state of the application internally. This can check if the application is able to work with it's dependencies. For ex: can the service connect to Postgres even though postgres is up? . An error in this service can indicate some sort of hard failure and can be hooked up to an outage dashboard. Ex: https://status.github.com/messages
3. Logging to a centralized system, like Graylog or Sentry. This will allow us to alert admins on outages. 


- We would like an efficient (fast) algorithm.
The algorithm seemed like a combinatorial problem. I don't see an obviously faster algorthm, although an "improved" implementation would avoid recursive structures and inner loops. Again, in the essence of time, I skipped this. 


# Paint batch optimizer service

## Purpose

This service provides solutions for the following problem.

Our users own paint factories. There are N different colors they can mix, and each color can be prepared "matte" or "glossy". So, you can make 2N different types of paint.

Each of their customers has a set of paint types they like, and customers will be satisfied if you have at least one of those types prepared. At most one of the types a customer likes will be a "matte".

Our user wants to make N batches of paint, so that:

There is exactly one batch for each color of paint, and it is either matte or glossy. For each customer, user makes at least one paint that they like. The minimum possible number of batches are matte (since matte is more expensive to make). This service finds whether it is possible to satisfy all customers given these constraints, and if it is, what paint types you should make. If it is possible to satisfy all your customers, there will be only one answer which minimizes the number of matte batches.

Input

Integer N, the number of paint colors,  integer M, the number of customers. A list of M lists, one for each customer, each containing: An integer T >= 1, the number of paint types the customer likes, followed by T pairs of integers "X Y", one for each type the customer likes, where X is the paint color between 1 and N inclusive, and Y is either 0 to indicate glossy, or 1 to indicated matte. Note that: No pair will occur more than once for a single customer. Each customer will have at least one color that they like (T >= 1). Each customer will like at most one matte color. (At most one pair for each customer has Y = 1). 

Output

The string "IMPOSSIBLE", if the customers' preferences cannot be satisfied; OR N space-separated integers, one for each color from 1 to N, which are 0 if the corresponding paint should be prepared glossy, and 1 if it should be matte.

## Usage

In the `app` directory you will see a small Python web service (`app.py`), a dependency list (`requirements.txt`) and a `Makefile`. The `Makefile` contains 2 targets: `build` that just installs the requirements into the current Python environment, and `run` which runs an example instance of the application.

The application has a primary endpoint at `/v1/`. When you make calls to this endpoint, you can send a JSON string as the argument "input". The JSON string contains three keys: colors, costumers and demands.

Examples:

http://0.0.0.0:8080/v1/?input={%22colors%22:1,%22customers%22:2,%22demands%22:[[1,1,1],[1,1,0]]}
IMPOSSIBLE

http://0.0.0.0:8080/v1/?input={%22colors%22:5,%22customers%22:3,%22demands%22:[[1,1,1],[2,1,0,2,0],[1,5,0]]}
1 0 0 0 0

## Limitations

None of our users produce more than 2000 different colors, or have more than 2000 customers. (1 <= N <= 2000 1 <= M <= 2000)
The sum of all the T values for the customers in a request will not exceed 3000.
