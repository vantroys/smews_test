Smews shell based test framework
================================

This repository allows building and checking smews.
The main script is the smews_test script. It takes one mandatory parameter,
the smews folder. 

When run, for each test suites in the test_suites folder it will generate all
possible configuration for all available smews targets and for an ipv4 and an
ipv6. For each configuration, it will:

1. Build smews
2. Run smews
3. Perform all test in the tests folder of the test suite
4. Perform all target dependent tests in the `tests/targets/<target>` folder of the test suite


Adding a new test suite
=======================

Test suites are available in the test_suites folder. All the tests of a test
suites are run *on the same running instance of smews*. Thus, for each
configuration, smews is built and run, then *all* the tests are performed
(including target dependent). Then, smews is killed and another configuration
is tested. You can tweak a test suite by following means:

1. You can develop specific applications that needs to be included in the smews
   build for your test suite to work. Any folder in the `<test_suite>/apps`
   folder will be considered as a smews application, copied to the smews folder
   and included in its build.

2. Any application (of the `smews/apps` folder) can be included in the build by
   listing its name in an `useapps` file in the test suite folder. One application
   per line.

3. A test suite can be designed *only* for given targets. To do so, list the targets
   in the `target` file, again one target per line.

4. A test suite can be designed for all targets *except* some. To do so, list
   the targets to exclude in the `notarget` file, one target per line. Warning,
   this is combined with the previous file. Thus, if a target is listed in the
   `notarget` file, it will *not* be tested, even if it is present in the
   `target` file.

5. By default, suites are tested for every possible combination of disable
   options. You can force an option to be disable or to be activated using
   `disable` and `nodisable` file. For example, if we suppose that the options
   that can be disabled in smews are `comet` and `post`, by default, the test
   suite will be run for each of the following build (the `disable=...` thing is
   the scons parameter for building smews):

   	* `disable=`
	* `disable=comet`
	* `disable=post`
	* `disable=comet,post`

   If the `disable` file contains `comet`, then the test will be with:

      	* `disable=comet`
	* `disable=comet,post`

   If the `nodisable` file contains `comet` and there is no `disable` file, the
   builds will be:

        * `disable=`
        * `disable=post`

Adding a new test in a test suite
=================================

A test can be any kind of executable for example a shell or python script (must
have the x bit though and the right usual `#!...` comment in first line):

*   It will be called with two parameters: the ip address of the tested smews and the target name,
*   It must exit with code 0 if test worked as expected or any positive value otherwise,
*   If the test is generic, it must be added in the tests folder,
*   If the test is target dependent, it must be added in the `<test_suite>/tests/target/<target>` folder,
*   The test are run in lexical sort order but all the generic tests will be
    performed before the target dependant tests
*   a `test_download.sh` script is available in the tools folder. You can consider
    that this script is in the PATH environment variable when your test is run,
    This script takes 3 parameters: the ip from which to download, the url and
    optionaly the expected http response code (200 is used if no value is
    given). The script returns true if all goes well.

Adding a target
===============

To add a target, you must implement 4 scripts in the `tools/<target>`
folder. The folder name is the one used by smews in its target folder
(for example linux,mbed_ethernet...).

1. `program` This script is called with the smews folder as the first
   parameter. When called, smews has been successfuly built for your target
   (and thus available in `$1/bin/<target>` with `$1` beeing the smews folder). You
   must program the device with the built image.
2. `is_alive` returns true if the smews server is still running. It is not used
   by the test framework but may be used by some tests to check if smews has crashed
3. `kill` stops the smews server on the target,
4. `run` run smews on the target

FAQ
===
`How can I use an application only for some targets ?`
> The only way is to create a dedicated test suite with the application and the corresponding targets listed in the `target` or `notarget`. If needed, you can simlink tests from other test suites to avoid re-writing them