====================
Testing using tester
====================

.. _tester-link

你可以使用我们的测试程序来测试实验。每个实验所属目录下都有一个可执行测试文件 `tester`

你可以从命令行在每个实验各自目录单独运行每个测试文件，以dns实验为例：

.. code-block:: text

    $ cd dns
    $ ./tester
    $ python3 tester


你可以带参数 `--help` 来查看帮助信息：

.. code-block:: text

    Testing onl program

    optional arguments:
      -h, --help            show this help message and exit
      -t testcase_index, --testcase testcase_index
                            running the ith testcase
      -d, --debug           use debug mode to show program output
      -l, --log             redirect testcase output to a log file
      -j, --json            generate a json file to store testing results


- 通过传递 `-t` 参数来运行指定测试用例，指定一个整数值 `N` 作为下标，针对第 `N` 个测试用例进行测试，测试用例存储在实验目录下的 `testcases.json` 文件中。
- 通过传递 `-d` 参数使用 `debug` 模式显示程序输出。
- 通过传递 `-l` 参数将测试用例输出重定向到日志文件，生成的日志文件在实验所属目录下的 `logs` 文件夹中，第 `N` 个测试用例对应生成一个 `testcaseN.log` 文件。

