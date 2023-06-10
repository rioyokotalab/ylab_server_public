## TOC
- [Login](https://github.com/rioyokotalab/ylab_server_public#login)
- [Batch job](https://github.com/rioyokotalab/ylab_server_public#batch-job)
- [Interactive job](https://github.com/rioyokotalab/ylab_server_public#interactive-job)
- [Changes from ex-lab cluster](https://github.com/rioyokotalab/ylab_server_public#changes-from-previous-lab-cluster)

## Login
- If you don't have account on Jumpcloud, please ask the server admins to create one for you
- If you haven't registered your ssh public key, please register it through JumpCloud

### Configurations on your computer

1. Setup `$HOME/.ssh/config` like below.
  - Replace the path to ssh public keys `~/.ssh/lab_rsa` with your own path.
  - Replace the user name with your own.

```
Host ylab
	Hostname login.rio.gsic.titech.ac.jp
	Identityfile ~/.ssh/lab_rsa
	ServerAliveInterval 15
	User ootomo.h
```

2. Login with `ssh` command
```bash
ssh ylab
```


## Batch job
### Example of a job script
```sh
#!/bin/bash
#YBATCH -r tr_1
#SBATCH -N 1
#SBATCH -J hello-world
#SBATCH --time=00:10:00

. /etc/profile.d/modules.sh
module load cuda

./a.out
```

*Hinadori cluster uses Slurm to schedule jobs but it uses Yokota lab original program to manage computing resources.*
Therefore, there are two kinds of specifiers, `#SBATCH` and `#YBATCH`.

### Specifiers for Slurm (#SBATCH)
- `#SBATCH -N [Number of nodes]` specifies the number of nodes you want to use.
- `#SBATCH -J [job name]` specifies the job name.
- `#SBATCH --output [output dir]/%j.out` specifies the directory to create output logs
- `#SBATCH --error [error dir]/%j.err` specifies the directory to create error logs
- `#SBATCH --time=71:00:00` specifies the maximum time this job can run for. Max of 7 days (168 hours).

### Specifiers for resource management
- `#YBATCH -r [resource group name]` specifies the computing resource that you want to use.
- `#YBATCH -p [job priority (low/verylow)]` specifies the priority of a job. Low priority jobs are executed after default priority jobs in the job queue. Once the low priority jobs are assigned to compute nodes, they are not evicted even if there are default priority jobs in the job queue.

You can see the list of resource groups with `yokota-rns` command.



(Output example)
```
Resource Name System (yokota-rns)

- Available resources
+-------------------+--------------+-----------------+-------------+---------------------------+----------+
|     resource_name |    partition | num CPU threads | Host memory |                       GPU | num GPUs |
+-------------------+--------------+-----------------+-------------+---------------------------+----------+
|             any_2 |          any |              16 |       90 GB |      psti_2/tr_2/titanv_2 |        2 |
|             any_1 |          any |               8 |       45 GB |      psti_1/tr_1/titanv_1 |        1 |
|          pascal_2 |       pascal |              16 |       90 GB |                    psti_2 |        2 |
|          pascal_1 |       pascal |               8 |       45 GB |                    psti_1 |        1 |
|          turing_2 |       turing |              16 |       90 GB |                      tr_2 |        2 |
|          turing_1 |       turing |               8 |       45 GB |                      tr_1 |        1 |
|           volta_1 |       turing |               8 |       45 GB |             teslav/titanv |        1 |
|           volta_2 |       turing |              16 |       90 GB |                    titanv |        2 |
|            psti_2 |       pascal |              16 |       90 GB | NVIDIA GeForce GTX 1080ti |        2 |
|            psti_1 |       pascal |               8 |       45 GB | NVIDIA GeForce GTX 1080ti |        1 |
|              tr_2 |       turing |              16 |       90 GB |   NVIDIA GeForce RTX 2080 |        2 |
|              tr_1 |       turing |               8 |       45 GB |   NVIDIA GeForce RTX 2080 |        1 |
|            trti_1 |         trti |               8 |       45 GB | NVIDIA GeForce RTX 2080ti |        1 |
|            trti_2 |         trti |              16 |       90 GB | NVIDIA GeForce RTX 2080ti |        2 |
|          titanv_2 |       titanv |              16 |       90 GB |            NVIDIA TITAN V |        2 |
|          titanv_1 |       titanv |               8 |       45 GB |            NVIDIA TITAN V |        1 |
|           a6000_2 |        a6000 |              28 |       60 GB |              NVIDIA A6000 |        2 |
|           a6000_1 |        a6000 |              14 |       30 GB |              NVIDIA A6000 |        1 |
|            teslav |       teslav |              16 |       90 GB |         NVIDIA Tesla V100 |        1 |
|        dgx-a100_1 |     dgx-a100 |              32 |      125 GB |               NVIDIA A100 |        1 |
|        dgx-a100_2 |     dgx-a100 |              64 |      250 GB |               NVIDIA A100 |        2 |
|        dgx-a100_4 |     dgx-a100 |             128 |      500 GB |               NVIDIA A100 |        4 |
|        dgx-a100_8 |     dgx-a100 |             256 |     1000 GB |               NVIDIA A100 |        8 |
|          titanrtx |     titanrtx |              32 |       64 GB |          NVIDIA TITAN RTX |        1 |
|       xeon-s-4215 |  xeon-s-4215 |              16 |       90 GB |                         - |        0 |
+-------------------+--------------+-----------------+-------------+---------------------------+----------+
```
If there are more than two GPUs on a node, this resource manager can split the computing resources by the number of GPUs and provide them for some number of jobs.

#### Recommendation 1: For someone who wants to use a node which you can use as soon as possible
`any_X` is good for you.
This resource group provides one of the available nodes which you can use immediately.

### Submit job script
Use `ybatch` command to submit a job script.
(Do not use `sbatch` command. `sbatch` cannot interpret `#YBATCH` specifier.)

```bash
ybatch job.sh
```

## Interactive job
Use `yrun` command to start an interactive job

```bash
yrun tr_1
```
The first argument is the name of the computing resource group you want to use.
You can see the list of resource groups with `yokota-rns` command.

## Changes from previous lab cluster
- Use `ybatch` and `yrun` command to submit jobs instead of `sbatch` and `srun`.
- You can not log in to a node via ssh even if your job is running on the node.
  - Currently it's still possible, but server admins will prohibit it later.
- Some module names are changed. Please run `module avail` and fix the module names that you load if necessary.
