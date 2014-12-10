This is my snakefile:
```
(snake-env)$ cat basic.snake
SANDWICHES = ['Jack.pbandj','Jill.pbandj']

rule all:
     input: SANDWICHES

rule clean:
     shell: "rm -f *.pbandj"

rule peanut_butter_and_jelly_sandwich_recipe:
     input: "kids/{name}", jelly="ingredients/jelly", pb="ingredients/peanut_butter"
     output: "{name}.pbandj"
     shell: "cat {input.pb} {input.jelly} > {output}"
```

`all` is a pseudo-rule - it only exists to establish the variable `SANDWICHES` as a target.
Since it is the first rule it will be run by default.

```
(snake-env)$ snakemake -s basic.snake
Provided cores: 1
Job counts:
    count	jobs
    2		peanut_butter_and_jelly_sandwich_recipe
    1		all
    3
rule peanut_butter_and_jelly_sandwich_recipe:
     input: kids/Jill, ingredients/peanut_butter, ingredients/jelly
     output: Jill.pbandj
1 of 3 steps (33%) done
rule peanut_butter_and_jelly_sandwich_recipe:
     input: kids/Jack, ingredients/peanut_butter, ingredients/jelly
     output: Jack.pbandj
2 of 3 steps (67%) done
rule all:
     input: Jack.pbandj, Jill.pbandj
3 of 3 steps (100%) done
```
We could have typed the targets directly:
```
(snake-env)$ snakemake -s basic.snake Jack.pbandj Jill.pbandj
Provided cores: 1
Job counts:
    count	jobs
    2		peanut_butter_and_jelly_sandwich_recipe
    2
rule peanut_butter_and_jelly_sandwich_recipe:
     input: kids/Jill, ingredients/jelly, ingredients/peanut_butter
     output: Jill.pbandj
1 of 2 steps (50%) done
rule peanut_butter_and_jelly_sandwich_recipe:
     input: kids/Jack, ingredients/jelly, ingredients/peanut_butter
     output: Jack.pbandj
2 of 2 steps (100%) done
```
But you cannot type variables as targets in Snakemake.
```
(snake-env)$ snakemake -s basic.snake snakemake -s SANDWICHES
MissingRuleException:
No rule to produce SANDWICHES.
```
You can only type in files and rules as targets.

`peanut_butter_and_jelly_sandwich_recipe` is an implicit rule - it uses wildcards to define its inputs and outputs. Can we run it?
```
(snake-env)$ snakemake -s basic.snake peanut_butter_and_jelly_sandwich_recipe
RuleException in line 6 of Snakefile:
Could not resolve wildcards in rule peanut_butter_and_jelly_sandwich_recipe:
name
```
Why didn't this work? Because Snakemake has no idea what you are trying to make.

A wildcard rule is a recipe, not a target. You might say, "hey why didn't it just look in the `kids` folder and then make a sandwich for each of them?" However, the `kids` directory was named just for neatness. Our rule could just as easily been:
```
rule peanut_butter_and_jelly_sandwich_recipe:
     input: "{name}", jelly="ingredients/jelly", pb="ingredients/peanut_butter"
     output: "{name}.pbandj"
     shell: "cat {input.pb} {input.jelly} > {output}"
```


So now any existing file in the filesystem is eligible for a sandwich! Exactly whose sandwich is it supposed to make if I run `peanut_butter_and_jelly_sandwich_recipe`?


In Snakemake, just as in Make:
*  Write re-usable wildcard rules based on transforming one type of file to another
*  Set variables to specify actual file targets
*  Write a pseudo-rule that uses the variable as input.

In Snakemake (but not Make) you have to write a pseudo-rule that uses the variable as input.

### How can we derive targets from existing source files?

Snakemake is Python, so we can simply use Python's `glob` function to read a directory contents and then transform the names into targets
```
KIDS = glob.glob('kids/*')
SANDWICHES = [os.path.basename(kid)+'.pbandj' for kid in KIDS]
```
You can try this with:
```
(snake-env)$ snakemake -s glob.snake
```
### How can we access the targets or sources from a list?
```
KIDS = [line.strip() for line in open("KidList.txt").readlines()]
SANDWICHES = [kid+'.pbandj' for kid in KIDS]
```
You can try this with:
```
(snake-env)$ snakemake -s listfile.snake
```
### How do I tell Snakemake which list of kids to process as a command line argument?
There are (at least) two ways we can accomplish this:
#### Use a configuration parameter
Snakemake autosets a global variable config, even if no configfile is loaded. This can be used to pass arguments to the snakefile.
```
#Usage: snakemake -s config.snake --config list=B
KIDS = [line.strip() for line in open(config.get("list")+".Kid.List.txt").readlines()]
SANDWICHES = [kid+'.pbandj' for kid in KIDS]

rule all:
     input: SANDWICHES

rule clean:
     shell: "rm -f *.pbandj"

rule peanut_butter_and_jelly_sandwich_recipe:
     input: "kids/{name}", jelly="ingredients/jelly", pb="ingredients/peanut_butter"
     output: "{name}.pbandj"
     shell: "cat {input.pb} {input.jelly} > {output}"

```

You can try this with:
```
(snake-env)$ snakemake -s config.snake --config list=A
```

#### Use a sentinel
The sentinel strategy, popular in Make, creates an output file as a sentinel, fake target from our list file and use a function that return a list of sandwiches as input.

This is easier if we maintain consistent naming conventions, i.e. we suffix lists as .List.txt. Our sentinel will be `Kid.List` to process a file called `Kid.List.txt`.
```
#Usage: snakemake -s listarg.snake A.Kid.List
import os
import fnmatch

def get_sandwiches(wildcards):
    #these get evaluated with wildcards from every rule
    #so be careful to open only the list file
    for wildcard in wildcards:
        if os.path.exists(wildcard+'.txt') and fnmatch.fnmatch(wildcard+'.txt', '*.List.txt'):
            kids = [line.strip() for line in open(wildcard+".txt").readlines()]
            sandwiches = [kid+'.pbandj' for kid in kids]
            return(sandwiches)
    return("")

rule listfile:
     input: get_sandwiches
     output: "{listfile}"
     shell: "touch {output}"

rule peanut_butter_and_jelly_sandwich_recipe:
     input: "kids/{name}", jelly="ingredients/jelly", pb="ingredients/peanut_butter"
     output: "{name}.pbandj"
     shell: "cat {input.pb} {input.jelly} > {output}"

rule clean:
     shell: "rm -f *.pbandj *List"
```
You can try this with:
```
(snake-env)$ snakemake -s listarg.snake A.Kid.List
```

