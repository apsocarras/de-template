# Data engineering project template

My personal fork of a **[`template`](https://github.com/josephmachado/data_engineering_project_template)** created by `@josephmachado.`

See **[`this post`](https://www.startdataengineering.com/post/data-engineering-projects-with-free-template/)** for a detailed explanation of the template.

### **Setup**

1. Copy local clone of this template to location of your next project: 
    ```shell 
    cp -Lr ~/<symlink_to_template> ~/projects/<new_project_name>
    cd ~/projects/<new_project_name>
    ```
2. Create new github repository with same name, add as remote, and push local directory to it.
2. In `.github/workflows/cd.yml`, change the value of the `TARGET` parameter after `/home/ubuntu/` to the name of the repository.
3. In `terraform/variable.tf`, change the default value of `repo_url` to your the repository url. 
4. Run the following commands in your project directory:

  ```shell
  # Local run & test
  make up # start the docker containers on your computer & runs migrations under ./migrations
  make ci # Runs auto formatting, lint checks, & all the test files under ./tests

  # Create AWS services with Terraform
  make tf-init # Only needed on your first terraform run (or if you add new providers)
  make infra-up # type in yes after verifying the changes TF will make

  # Wait until the EC2 instance is initialized, you can check this via your AWS UI
  # See "Status Check" on the EC2 console, it should be "2/2 checks passed" before proceeding
  # Wait another 5 mins, Airflow takes a while to start up

  make cloud-airflow # this command will forward Airflow port from EC2 to your machine and opens it in the browser
  # the user name and password are both airflow

  make cloud-metabase # this command will forward Metabase port from EC2 to your machine and opens it in the browser
  # use https://github.com/josephmachado/data_engineering_project_template/blob/main/env file to connect to the warehouse from metabase
  ```

### **Infrastructure Overview**
![DE Infra](/assets/images/infra.png)
![Project structure](/assets/images/proj_1.png)
![Project structure - GH actions](/assets/images/proj_2.png)

**Creating Database Migrations**
```shell
make db-migration # enter a description, e.g. create some schema
# make your changes to the newly created file under ./migrations
make warehouse-migration # to run the new migration on your warehouse
```

For the [continuous delivery](https://github.com/josephmachado/data_engineering_project_template/blob/main/.github/workflows/cd.yml) to work, set up the infrastructure with terraform, & defined the following repository secrets. You can set up the repository secrets by going to `Settings > Secrets > Actions > New repository secret`.

1. **`SERVER_SSH_KEY`**: We can get this by running `terraform -chdir=./terraform output -raw private_key` in the project directory and paste the entire content in a new Action secret called SERVER_SSH_KEY.
2. **`REMOTE_HOST`**: Get this by running `terraform -chdir=./terraform output -raw ec2_public_dns` in the project directory.
3. **`REMOTE_USER`**: The value for this is **ubuntu**.

### Tear down infra

After you are done, make sure to destroy your cloud infrastructure.

```shell
make down # Stop docker containers on your computer
make infra-down # type in yes after verifying the changes TF will make
```