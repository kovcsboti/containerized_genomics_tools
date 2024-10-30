This subdirectory contains the necessary files build and manage Docker and Apptainer containers for the fastp tool.

# Usage

## Running `fastp` from the Docker container
First pull the Docker container using `make pull`. You need to have Docker permissions to do this or to run the Docker container. Then `docker run deptmetagenom/fastp` runs the tool and displays its command-line parameters.

Alternatively, if you don't have other `fastp` Docker images on the local machine, tag `deptmetagenom/fastp` as `fastp` to save typing:
```
docker tag deptmetagenom/fastp fastp
```
Then the command is `docker run fastp`.

To run fastp on an input file, the directory containing the input file on the host machine must be mounted on the Docker container using the `docker run -v /path/to/host:/container/directory` switch. For example, if the input file is in the current working directory (i.e. the directory active in the terminal when running the `docker run` command) or one of its subdirectories, then you would mount the current working directory on the container like this:
```
docker run -it --rm --user $(id -u):$(id -g) -v $PWD:$PWD -w $PWD fastp -i test/in/R1.fq -o R1out.fq
```

This will run `fastp` on the input file located in the `test/in` subdirectory of the current working directory and write the output to the file `R1out.fq` in the current working directory. In addition, the html and json output files generated by `fastp`, which are called `fastp.html` and `fastp.json` by default, are written into the current working directory as well. The name and path of these files can be changed using the `-h` and `-j` arguments respectively.

The Docker container will need write permissions to write the output file to the specified path and the html and json output to the current working directory or to the directory specified using `-h` and `-j`. The recommended way to ensure that these files can be written is to run the containerised application as the current user using the `--user` argument as specified above. This will give write and read permissions identical to the current user's within the mounted directory (which is the working directory) to the `fastp` tool.

## Running `fastp` from the Apptainer container
You don't need any special permissions to run the Apptainer container, nor to mount any directories. Run it like this:

```
apptainer run fastp_0.23.4.sif -i test/in/R1.fq -o R1out.fq
```

## `make pull`
Pull the latest version of `fastp` from the institute's Docker Hub.

## `make build`
Builds a Docker image on the local machine using the `Dockerfile`.

## `make push`
First builds the Docker image by running `make build`, and then pushes it to the institute's Docker Hub repository. When uploading the file, Docker asks for the repository's user name and password. Either specify your own Docker Hub account when running this, i.e. `make push DOCKER_USER=myaccount`, or if you want to push to the institute's repository and don't have the credentials, ask the admins (Laczkó Levente, Pethő Gergely) for it.

## `make build_apptainer`
Runs `make build push`, then creates an Apptainer image by converting the Docker image.

## `make test`
Test the Docker image and then the Apptainer image by running them on a few input files and comparing the output generated by the containerised tools to the reference output. If the corresponding reference output differs from any of the generated outputs, the test fails.

## Updating the containers to a new version of `fastp`
If a new version of `fastp` is released, override or adjust the value assigned to the `FP_VERSION` variable in the Dockerfile, then run `make build_apptainer`, followed by `make test`. If `make test` fails, this can mean either that building either the Docker or the Apptainer container failed, or that the output of `fastp` for the input file has changed compared to the earlier version of `fastp` e.g. because of added functionality or bug fixes. If the output for the updated container appears correct, copy the generated outputs into the ref subdirectory to treat them as the new reference outputs for testing. Rerun `make test` to make sure.