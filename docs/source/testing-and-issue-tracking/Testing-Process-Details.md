# Attachments

## A. Identification of Compilers Supported by Kokkos

The following list is taken from file kokkos/README in the Kokkos Github repository. (15Feb18)

* Primary tested compilers on X86 are:
    * GCC 4.8.4
    * GCC 4.9.3
    * GCC 5.1.0
    * GCC 5.3.0
    * GCC 6.1.0
    * GCC 7.3.0
    * GCC 8.1.0
    * Intel 17.1.043
    * Intel 17.4.196
    * Intel 18.0.128
    * Clang 4.0.0 for CPU
    * Clang 10.0.0 for CUDA (CUDA Toolkit 11.0.1)
    * PGI 18.10
    * NVCC 7.0 for CUDA (with gcc 4.8.4)
    * NVCC 7.5 for CUDA (with gcc 4.8.4)
    * NVCC 8.0.44 for CUDA (with gcc 5.3.0)

* Primary tested compilers on Power 8 are:
    * GCC 5.4.0 (OpenMP,Serial)
    * IBM XL 13.1.5 (OpenMP, Serial) (There is a workaround in place to avoid a compiler bug)
    * NVCC 8.0.44 for CUDA (with gcc 5.4.0)
    * NVCC 9.0.103 for CUDA (with gcc 6.3.0)

* Primary tested compilers on Intel KNL are:
    * GCC 6.2.0
    * Intel 17.2.174 (with gcc 4.9.3)
    * Intel 18.0.128 (with gcc 4.9.3)

* Other compilers working:
    * X86:
    * Cygwin 2.1.0 64bit with gcc 4.9.3

* Known non-working combinations:
    *  Power8:
    *  Pthreads backend

Primary tested compiler are passing in release mode with warnings as errors.

They also are tested with a comprehensive set of backend combinations:
    ```OpenMP, Pthreads, Serial, OpenMP+Serial, ...```

We are using the following set of flags:
    ```GCC:   -Wall -Wshadow -pedantic -Werror -Wsign-compare -Wtype-limits```

----

## B.  File:  kokkos-promotion.txt

Summary:

- Step 1: Testing Kokkos itself using test_all_sandia

- Step 2: Testing of Kokkos integrated into Trilinos via ATDM configurations as well as against LAMMPS and SPARC. 

- Step 3: Locally update CHANGELOG, merge into master, edit config/master_history.txt

- Step 4: Locally snapshot new master into corresponding Trilinos kokkos-promotion branch and issue pull-request against Trilinos' develop branch.

- Step 5: Push local Kokkos master to GitHub after Trilinos pull request was accepted.

Steps 1, 2, and 4 include testing that may fail. These failures must be fixed either by pull requests to Kokkos develop, or by creating a new Trilinos branch for parts of Trilinos that must be updated. This is what usually takes the most time.

// ----------------------------------------------------------------------------------------------------- //

Step 1: The following should be repeated on enough machines to cover all
supported compilers. Those machines are:

    kokkos-dev
    kokkos-dev-2

  1.1. Clone kokkos develop branch (or just switch to it)

    git clone -b develop git@github.com:kokkos/kokkos.git
    cd kokkos

  1.2. Create a testing directory

    mkdir testing
    cd testing

  1.3. Run the test_all_sandia script with no options to test all compilers

    nohup ../config/test_all_sandia &
    tail -f nohup.out                   # to watch progress

// ----------------------------------------------------------------------------------------------------- //

Step 2: Build and Test

  2.1. Build and test Trilinos with 4 different configurations; run scripts for White and Shepard that are provided in kokkos/scripts/trilinos-integration. These scripts load their own modules/environment, so don't require preparation. You can run all four at the same time, use separate directories for each.

    mkdir serial
    cd serial
    nohup KOKKOS_PATH/scripts/trilinos-integration/
                    shepard_jenkins_run_script_serial_intel &

  2.2. Compare the compile errors and test failures between updated and pristine versions. There may be compilation failures that happen in both, tests that fail in both, and there may be tests that only fail sometimes (thus, rerun tests manually as needed).

// ----------------------------------------------------------------------------------------------------- //

Step 3: This step should be run on kokkos-dev

  3.1. If you don't have a GitHub token already, generate one for yourself (this will give you TOKEN):

    https://github.com/settings/tokens

  3.2. Get a clean copy of the Kokkos develop branch

    git clone -b develop git@github.com:kokkos/kokkos.git
    cd kokkos

  3.3. Generate the initial changelog. Use the most recent tag as OLDTAG (`git tag -l` can show you all tags). The NEWTAG is the new version number, e.g. "2.04.00". RUN THIS OUTSIDE THE KOKKOS SOURCE TREE!

     module load ruby/2.3.1/gcc/5.3.0
     gitthub_changelog_generator kokkos/kokkos --token TOKEN --no-pull-requests --include-labels 'InDevelop' --enhancement-labels 'enhancement,Feature Request' --future-release 'NEWTAG' --between-tags 'NEWTAG,OLDTAG'
     cat CHANGELOG.md

  3.4. Manually cleanup and commit the change log. Pushing to develop requires Owner permission.
       (Copy the new section from the generated CHANGELOG.md to KOKKOS_PATH/CHANGELOG.md)
       (Make desired changes to CHANGELOG.md to enhance clarity (remove issues not noteworthy))
       (Commit and push the CHANGELOG.md to develop)

  3.5. Merge develop into master. DO NOT FAST-FORWARD THE MERGE!!!!

    (From kokkos directory):
    git checkout master
    git merge --no-ff origin/develop

  3.6. Update the tag in kokkos/master_history.txt

    Tag description: MajorNumber.MinorNumber.WeeksSinceMinorNumberUpdate
    Tag field widths: #.#.##
    date description: month:day:year
    date field widths: ##:##:####
    master description: SHA1 of previous master commit (use `git log`?)
    develop description: SHA1 of merged develop branch
    SHA1 field width: ######## (8 chars)
    
    # Append to master_history.txt:
    
    tag:  2.03.13    date: 07:27:2017    master: da314444    develop: 29ccb58a
    
    git commit --amend -a


  3.7. Create the new tag:

    git tag -a #.#.##
    
      (type the following into the tag message (same as for step 4.3))
      tag: #.#.##
      date: mm/dd/yyyy
      master: sha1
      develop: sha1

  3.8. DO NOT PUSH YET !!!

// ----------------------------------------------------------------------------------------------------- //

Step 4: This step can be done on any SEMS machine (e.g. kokkos-dev). Actually, the checkin step requires lots of disk space and RAM. Use ceerws1113 if you have access to it.

  4.1 Clone the Trilinos corresponding branch (or just switch to it)

    git clone -b develop git@github.com:trilinos/Trilinos.git
    TRILINOS_PATH=$PWD/Trilinos

  4.2 Snapshot Kokkos into Trilinos kokkos-promotion branch - this requires python/2.7.9 and that both Trilinos and Kokkos be clean - no untracked or modified files. Run the following outside the Kokkos and Trilinos source trees.

    module load sems-python/2.7.9
    python KOKKOS_PATH/scripts/snapshot.py KOKKOS_PATH TRILINOS_PATH/packages

  4.3. Issue pull request against Trilinos develop branch.

  4.4. If there are failures, fix and backtrack. Otherwise, go to next step

// ----------------------------------------------------------------------------------------------------- //

Step 5: Push Kokkos master to GitHub (requires Owner permission).

    cd KOKKOS_PATH
    git push --follow-tags origin master
