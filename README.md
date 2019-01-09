# vmcluster

To configure and run a local VM cluster using Vagrant. It is based on the CEDA Ccentos6 release. The following extra software will be installed:
- Slurm
- MPICH
- Singularity
- Docker

## Setup:

edit Vagrantfile to change:
- N_COMP_NODES to the number of require compute nodes. The master node will always be created, and configured to run batch jobs.
- N_CORES the number of cores per node.
- LOCAL_NFS to a local empty directory. The cluster will nfs mount this directory at boot on all nodes under /vagrant. 

To boot the cluster
```
cd vmcluster
vagrant up
```

To log on
```
vagrant ssh master
```

To shut it down
```
vagrant down
```

To remove it completely
```
vagrant destroy
```


To see if slurm is OK:
```
scontrol show node
```


Test the cluster with
```
srun -N2 -n4 --mpi=pmi2 $HOME/mpi_hello_world
```

This should produce output similar to:

```
Hello world from processor master, rank 2 out of 4 processors
Hello world from processor master, rank 3 out of 4 processors
Hello world from processor compute-node-1, rank 0 out of 4 processors
Hello world from processor compute-node-1, rank 1 out of 4 processors
```

Note: --mpi=pmi2 is required for slurm batch commands to ensure MPICH is used correctly