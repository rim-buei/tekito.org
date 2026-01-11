シグナルをブロックすると fork() 後の子プロセスもブロックされた状態で生成される
##############################################################################

:date: 2013-12-03 23:30
:slug: signal-state-will-be-inherited-by-child-process
:tags: C/C++, Linux

シグナルをブロックした状態で fork() すると、子プロセスもシグナルをブロックした状態で生成されます。

.. code-block:: c++

                #include <cstdio>
                #include <cstdlib>
                #include <csignal>
                #include <unistd.h>
                #include <iostream>

                int main(void) {
                    sigset_t blockmask;
                    sigemptyset(&blockmask);
                    sigaddset(&blockmask, SIGTERM);
                    sigprocmask(SIG_BLOCK, &blockmask, NULL);

                    pid_t pid = fork();
                    if (pid < 0) {
                        perror("fork");
                        exit(EXIT_FAILURE);
                    } else if (pid == 0) {
                        /* child */
                        sigset_t checkmask;
                        sigemptyset(&checkmask);
                        sigprocmask(0, NULL, &checkmask);
                        if (sigismember(&checkmask, SIGTERM)) {
                            std::cout << "SIGTERM is blocked." << std::endl;
                        }
                    } else {
                        /* parent */
                        sleep(1);
                    }

                    return 0;
                }

出力結果:

.. code-block:: bash

                $ g++ block_sigterm.cpp -o block_sigterm
                $ ./block_sigterm
                SIGTERM is blocked.


fork() したあとに exec() 系の関数を呼び出したりすると、ブロックされた状態で処理が継続するようなので思わぬ結果になります。

fork() 前に pending 状態のシグナルは子プロセスには引き継がれません。
