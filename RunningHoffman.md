#<a name="hoffman">Using Hoffman </a>

## Table of contents
1. [Interactive Session](#qrsh)
2. [Submitting a job](#qsub)
3. [Miscellaneous](#misc)
4. [Job Array](#jobarray)


###<a name ="qrsh">Interactive Session</a>

If you want to run everything live in the terminal, you want an interactive session. This is nice when you have an environment you set up and you want to see if things work now. There is no waiting for the job to run like when you submit a job on hoffman2. 

**Time max**: you can request up to 24 hours on general computing nodes. If you Group has it's own nodes you can request up to 2 weeks of time: 336 hours.

[Guidelines](https://www.hoffman2.idre.ucla.edu/computing/interactive-session/)

Example: You want to request an interactive session with 4GB of memory for 8 hours.

```
qrsh -l h_rt=8:00:00,h_data=4G
```

Example: You want to run a job on multiple cores (for parallel processing)

```
qrsh -l h_rt=8:00:00,h_data=4G -pe shared 4
```

Example: You want to run a job for 200 hours. Use the high-performance node. 

```
qrsh -l h_rt=8:00:00,h_data=4G,highp
```

 


###<a name ="qsub">Submit a Job</a>


If you need to run a job for a long time and you don't want to wait for an interative session, you can submit a job. There are multiple parameters you can set to get what you want. For example, you may want to receive an email when the job starts or stops or fails.


First step to a job, you need to write a bash script to run the job. Bash scripts have extension ".sh" . 

[Guidelines](https://www.ccn.ucla.edu/wiki/index.php/Hoffman2:Submitting_Jobs)

Example: ```cwd``` means I want to run the job from the directory I am currently submitting it from. ```-M``` means I want to receive an email to the email registered to my username. Replace 'yourusername' with your hoffman2 username. ```-m beas``` defined when I want to be emailed: if the job **b**egins, if the job **e**nds. Check out the other parameters at [Hoffman2 wiki](https://www.ccn.ucla.edu/wiki/index.php/Hoffman2:Submitting_Jobs)


```{bash}
qsub -cwd -V -N Dump1_5 -l h_data=4G,h_rt=24:00:00 -M yourusername -m beas -b y "./fastq_dump_script.sh"
```

**When you need specific modules/ Running Python/ R** 

[Guidelines](https://www.hoffman2.idre.ucla.edu/computing/modules/#How_to_use_the_module_command_in_scripts_for_batch_execution)
You will need to include a module load command at the top just like with interactive Hoffman2.

example_script.sh

```
#!/bin/bash
. /u/local/Modules/default/init/modules.sh
module load python/anaconda3
my-script.py argument1 argument2
```


###<a name ="miscellaneous">Miscellaneous Skills</a>

**Check sob status for all your jobs**

```
qstat -u username
```


**View Command for job id**

```
qstat -j <i>jobid</i>
```

**Check how full your directories are, and what is available, aka My quota**

```
myquota -u username
```		

###<a name ="jobarray">Running a job array or Batch job</a>

Sometimes you want to do the same task to 1000 different files with the same parameters or different parameters. You want it to happen fast. Turns out you can submit 1 job that will release 1000 tasks nearly simultaneously so that you can have several 100 jobs running at the same time. This can turn a 1000 minute task into a 2 minute task. 
 
####Step 1: Prepare command parameters files. Each "job" needs it's own file that contains the parameters

Each parameters should be named with a number that represents a job number between 1 and the number of jobs you want to run. For example, data1.in, data2.in, data3.in.

[Guidelines](https://www.hoffman2.idre.ucla.edu/computing/job_arrays/)

**single arguments**
data1.in looks like a simple text file:

```
argument1


```

**multiple arguments**
data1.in looks like a simple text file

```
argument1 argument2


```

#### Step 2: Write a script that will match up the task you want to run to the appropriate parameter file datai.in

The task itself can be a python script ```yourscript.py``` or a bash ```script yourscript.sh```

Example: I put all the datai.in files in a directory called PARAMS

**single arguments**

run_task.sh

```
#!/bin/sh
. /u/local/Modules/default/init/modules.sh
module load python/anaconda3
python sep_v_regions.py < PARAMS/param$SGE_TASK_ID.in
```

or more elegantly, printing the task number and checking if the file exists

```
#!/bin/sh
echo “Task id is $SGE_TASK_ID”
if [ -e PARAMS/data$SGE_TASK_ID.in ]; then
	. /u/local/Modules/default/init/modules.sh
	module load python/anaconda3
	python sep_v_regions.py < PARAMS/param$SGE_TASK_ID.in
fi
```



**multiple arguments**

run_task.sh

```
#!/bin/bash

while read -r arg_1 arg_2 arg_3; do 
    ./task.sh "${arg_1}:${arg_2},${arg_3}"
done < PARAMS/data$SGE_TASK_ID.in
```

####Step 3: Submit the job array

Running jobs 1 to 10 I indicate the job numbers (which match the parameter file numbers) with ```-t lower-upper:interval```

When submitting the job array submit the instruction as follows
```
qsub -cwd -V -N First10 -l h_data=1G,time=10:00:00 -M briscoel -m beas -b y -t 1:10 "./run_task.sh"
```
