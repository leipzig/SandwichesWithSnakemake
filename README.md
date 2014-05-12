This is my snakefile:
```
(raboso-env)[leipzigj@raboso snakemaketutorial]cat Snakefile 
SANDWICHES = ['jack.pbandj','jill.pbandj']

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

```
(raboso-env)[leipzigj@raboso snakemaketutorial]$snakemake all
Provided cores: 1
Job counts:
    count	jobs
    2		peanut_butter_and_jelly_sandwich_recipe
    1		all
    3
rule peanut_butter_and_jelly_sandwich_recipe:
     input: kids/jill, ingredients/peanut_butter, ingredients/jelly
     output: jill.pbandj
1 of 3 steps (33%) done
rule peanut_butter_and_jelly_sandwich_recipe:
     input: kids/jack, ingredients/peanut_butter, ingredients/jelly
     output: jack.pbandj
2 of 3 steps (67%) done
rule all:
     input: jack.pbandj, jill.pbandj
3 of 3 steps (100%) done
```
We could have typed the targets directly:
```
(raboso-env)[leipzigj@raboso snakemaketutorial]$ snakemake jack.pbandj jill.pbandj
Provided cores: 1
Job counts:
    count	jobs
    2		peanut_butter_and_jelly_sandwich_recipe
    2
rule peanut_butter_and_jelly_sandwich_recipe:
     input: kids/jill, ingredients/jelly, ingredients/peanut_butter
     output: jill.pbandj
1 of 2 steps (50%) done
rule peanut_butter_and_jelly_sandwich_recipe:
     input: kids/jack, ingredients/jelly, ingredients/peanut_butter
     output: jack.pbandj
2 of 2 steps (100%) done
```
But you cannot type variables as targets in Snakemake.
```
(raboso-env)[leipzigj@raboso snakemaketutorial]$ snakemake SANDWICHES
MissingRuleException:
No rule to produce SANDWICHES.
  File "/nas/is1/bin/raboso-env/lib/python3.3/functools.py", line 275, in wrapper
```
You can only type in files and rules as targets.

`peanut_butter_and_jelly_sandwich_recipe` is an implicit rule - it uses wildcards to define its inputs and outputs. Can we run it?
```
(raboso-env)[leipzigj@raboso snakemaketutorial]$ snakemake peanut_butter_and_jelly_sandwich_recipe
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
