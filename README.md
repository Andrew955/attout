# attout

If you have to run bunches of psmatch2 and output them, this will be a good choice instead of endless ctrl+c/ctrl+v. Meanwhile, you'd better use the loop i provided below. It will my pleasure to hear you advice.

**Description：**
- Use scalars to store results after psmatch2
- Put aforementioned scalars into matrix
- Logout matrix into xml or any other filetype



![output example](https://img-blog.csdnimg.cn/20200307215320264.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3F5ZjFtZA==,size_16,color_FFFFFF,t_70)

## Code

1. Use your data & Prepare for psmatch2.

```
clear all
use "data.dta"
set seed 10101
gen ranorder=runiform()
sort ranorder
```

2. Define global variables to simplify the code and make further adjustments easier.

```
global PS "x1 x2 x3 x4 x5" //define indepvars
global Y "dif_y1 dif_y2 dif_y3" //define outcome varlist
global Y_out "dif_y1, dif_y2, dif_y3" //define outcome varlist to output, you have to add comma between any of them.
```

3. Define you own stata program attout.

```
cap program drop attout
program define attout
version 1.0 //2020.03.04
args var name //define two macronames, eg. var=dif_y1 & name=dif_y1
	sca `var'_1 = r(att) //put att into scalar
	sca `var'_2 = r(seatt) //put seatt into scalar
	count if (_treat==1)&(_support==1) //count the treated
	sca `var'_3 = r(N) //put the treated number into scalar
	count if (_treat==0)&(_support==1)
	sca `var'_4 = r(N)
	count if _support==1
	sca `var'_5 = r(N)
	sca `var'_6 = `var'_1/`var'_2 //calculate the t-value
	mat `var' = (`var'_1 \ `var'_2 \ `var'_3 \ `var'_4 \ `var'_5 \ `var'_6) //put all scalars into a matrix
	mat colnames `var' = `name' //give the matrix colnames
	mat rownames `var' = ATT SE Treated Untreated N T-stat //give the matrix rownames
end
```

4. Psmatch2 loop.

```
foreach v of varlist $Y{ //aforedefined outcome varlist
psmatch2 treat $PS, kernel com quiet outcome(`v') //you can rewrite your own psmatch2 code.
attout `v' `v' //apply aforedefined program "attout"
}
```

5. Logout.

```
logout, save(myfile) dec(3) excel replace: matlist [$Y_out] ,nohalf //aforedefined outcome varlist
```

## One more thing
- **Environment**: Stata 16.0 (other versions may also work)
- **To do**: Since the matrix does not support inputting string variables, i can not add any "\*" to represent significance level. You can do it manually since the program has provided the t-value.
- **Contact**: You can contact me via andrew955#qq.com(@). All remaining errors are my own.
