# How to organize npm script subtask with pre script

## my first npm scripts

package.json

```
{
  "script": {
    "pre": "echo \">>>>>>  running pre ...\" && npm-run-all --serial pre:preA pre:preB",
    "pre:preA": "echo \">>>>>>  running pre:preA ...\" && exit 0",
    "pre:preB": "echo \">>>>>>  running pre:preB ...\" && exit 0",
    "build": "echo \">>>>>>  running build ...\" && npm-run-all --serial pre build:*",
    "prebuild:modA": "npm run pre:preA",
    "build:modA": "echo \">>>>>>  running build:modA ...\" && exit 0",
    "prebuild:modB": "npm run pre:preB",
    "build:modB": "echo \">>>>>>  running build:modB ...\" && exit 0"
  }
}
```

when I run sub mod build, everything goes well

```
 npm run build:modA

> npm-script-subtask-pre@1.0.0 prebuild:modA
> npm run pre:preA


> npm-script-subtask-pre@1.0.0 pre:preA
> echo ">>>>>>  running pre:preA ..." && exit 0


> npm-script-subtask-pre@1.0.0 build:modA
> echo ">>>>>>  running build:modA ..." && exit 0

">>>>>>  running build:modA ..."
```

```
npm run build:modB

> npm-script-subtask-pre@1.0.0 prebuild:modB
> npm run pre:preB


> npm-script-subtask-pre@1.0.0 pre:preB
> echo ">>>>>>  running pre:preB ..." && exit 0

">>>>>>  running pre:preB ..."

> npm-script-subtask-pre@1.0.0 build:modB
> echo ">>>>>>  running build:modB ..." && exit 0

">>>>>>  running build:modB ..."
```

but if I run build, the pre subtask run 2 times (if there are more subtask, it will run more times)

```
npm run build

> npm-script-subtask-pre@1.0.0 build
> echo ">>>>>>  running build ..." && npm-run-all --serial pre build:*

">>>>>>  running build ..." 

> npm-script-subtask-pre@1.0.0 pre
> echo ">>>>>>  running pre ..." && npm-run-all --serial pre:preA pre:preB

">>>>>>  running pre ..." 

> npm-script-subtask-pre@1.0.0 pre:preA
> echo ">>>>>>  running pre:preA ..." && exit 0

">>>>>>  running pre:preA ..." 

> npm-script-subtask-pre@1.0.0 pre:preB
> echo ">>>>>>  running pre:preB ..." && exit 0

">>>>>>  running pre:preB ..." 

> npm-script-subtask-pre@1.0.0 prebuild:modA
> npm run pre:preA


> npm-script-subtask-pre@1.0.0 pre:preA
> echo ">>>>>>  running pre:preA ..." && exit 0

">>>>>>  running pre:preA ..."

> npm-script-subtask-pre@1.0.0 build:modA
> echo ">>>>>>  running build:modA ..." && exit 0

">>>>>>  running build:modA ..."

> npm-script-subtask-pre@1.0.0 prebuild:modB
> npm run pre:preB


> npm-script-subtask-pre@1.0.0 pre:preB
> echo ">>>>>>  running pre:preB ..." && exit 0

">>>>>>  running pre:preB ..."

> npm-script-subtask-pre@1.0.0 build:modB
> echo ">>>>>>  running build:modB ..." && exit 0

">>>>>>  running build:modB ..."
```

## Question

how to organize npm script:

* when run build:modA

```
# run buid:modA's pre script
run pre:preA

# then run build:modA it self
run build:modA
```

* when run build,  
```
# run build's pre script
run pre:A
run pre:B

# then run build's sub task

# below here is important part! (also be the question point): 

# before run build:modA
# how to detect that the pre:preA already run in this workflow instance, so we can skip it ?
run build:modA

# before run build:modB
# also how to skip pre:preB
run build:modB

```

is there a way to organize my subtask, in a graceful style ?

## Addition

IF we delte top build's pre script, It goes well. 

BUT....IF... 

The situation becomes more complicated: subtask is more, and prepare task is cross-over, there will be also duplicated pre task run without necessity

this is a example:

modA needs preA  
modB needs preB  
modC needs preA also  
modD needs preA and preB  

```
{
  "script": {
    "pre": "echo \">>>>>>  running pre ...\" && npm-run-all --serial pre:preA pre:preB",
    "pre:preA": "echo \">>>>>>  running pre:preA ...\" && exit 0",
    "pre:preB": "echo \">>>>>>  running pre:preB ...\" && exit 0",
    "build": "echo \">>>>>>  running build ...\" && npm run build:*",
    "prebuild:modA": "npm run pre:preA",
    "build:modA": "echo \">>>>>>  running build:modA ...\" && exit 0",
    "prebuild:modB": "npm run pre:preB",
    "build:modB": "echo \">>>>>>  running build:modB ...\" && exit 0",
    "prebuild:modC": "npm run pre:preA",
    "build:modC": "echo \">>>>>>  running build:modC ...\" && exit 0",
    "prebuild:modD": "npm run pre:preA & npm run pre:preB",
    "build:modD": "echo \">>>>>>  running build:modD ...\" && exit 0",
  }
}
```
