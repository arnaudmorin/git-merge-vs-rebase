# Git Merge VS Rebase conflicts example

In this repo, there are two branches:
 * stein
 * queens

Stein contains the following commits:

 * 1ff2641 (stein) G
 * cf24853 F
 * 3fcdfa0 E
 * cb8c71a D
 * 849ef26 C
 * 372c155 B
 * edc8ed7 A

Queens contains the following commits:

 * 2624c16 (queens) F
 * 5e3521b B
 * edc8ed7 A

So the common ancestor is A.

B has been cherry-picked from stein to queens, without conflicts.

F has also been cherry-picked but with conflicts (solved, of course).

## Merging

First, create a branch from queens

```bash
git checkout -b merged origin/queens
```

Then merge the stein branch into your branch
```bash
git merge origin/stein
```

You will have a conflict to solved.

This is due to the fact that F was cherry-picked on queens branch.

You can solve the conflict easily.

You should finally obtain something with a diff to stein:

```bash
git diff stein
```

```bash
diff --git a/libvirt.pp b/libvirt.pp
index c37b5ea..bd5eabe 100644
--- a/libvirt.pp
+++ b/libvirt.pp
@@ -278,14 +278,20 @@ class nova::compute::libvirt (
     validate_legacy(String, 'validate_string', $libvirt_cpu_model)
     nova_config {
       'libvirt/cpu_model': value => $libvirt_cpu_model;
+      'libvirt/cpu_model_extra_flags': value => $libvirt_cpu_model_extra_flags;
     }
   } else {
     nova_config {
       'libvirt/cpu_model': ensure => absent;
+      'libvirt/cpu_model_extra_flags': ensure => absent;
     }
     if $libvirt_cpu_model {
       warning('$libvirt_cpu_model requires that $libvirt_cpu_mode => "custom" and will be ignored')
     }
+
+    if $libvirt_cpu_model_extra_flags {
+      warning('$libvirt_cpu_model_extra_flags requires that $libvirt_cpu_mode => "custom" and will be ignored')
+    }
   }
 
   if $libvirt_cpu_mode_real != 'none' {

```

## Rebasing

First, create a branch from queens

```bash
git checkout -b rebased origin/queens
```

Now rebase it on top of stein:

```bash
git rebase -i origin/stein
```

Now, we have two choices:

 * Dont cherry-pick the commit that are already included (if we know that F is included into stein, we can remove the commit from beeing applied).
 * Or keep the cherry-picking of the F commit because we are not sure if it's applied in stein.

If we are not sure, we can keep it, but it will conflict.

We can solve the conflict easily.

Either way, the result will be the same:

```bash
git diff origin/stein
# empty
```

No diff to stein :p

And a clean history!


# Explanation

Read this: https://git-scm.com/docs/merge-strategies

Part of it:

```
With the strategies that use 3-way merge (including the default, 'recursive'), if a change is made on both branches, but later reverted on one of the branches, that change will be present in the merged result; some people find this behavior confusing. It occurs because only the heads and the merge base are considered when performing a merge, not the individual commits. The merge algorithm therefore considers the reverted change as no change at all, and substitutes the changed version instead.
```


