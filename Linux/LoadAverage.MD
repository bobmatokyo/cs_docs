【Linux】Load Averageに関するいろは

システム全体の実行待ちプロセス数の平均値

背景
CPU使用率だったりAverageLoadだったりを見てサーバの負荷度だったりを調査する必要があってそれについていろいろ疑問だった点をまとめました。
間違い等あればご指摘お願いします。

ボトルネック調査の順で言うととりあえず最初にやるかなって所を書いてます

1.CPU使用率 ★対象
2.メモリ使用量
3.ディスクI/O
4.TCPコネクション数

GitHub - torvalds/linux: Linux kernel source tree
https://github.com


Load Averageって？
ロードアベレージはシステム全体の負荷状況を表す指標。
「1CPUにおける単位時間あたりの実行待ちとディスクI/O待ちのプロセスの数」で表される。

Linuxカーネルはプロセス１つごとにプロセスディスクリプタを持っている。
Linuxでのプロセスディスクリプタはtask_struct構造体という名前になっていて、プロセスに関する全ての情報が入っています。

以下はtask_structの実装の一部です。

include/linux/sched.h
struct task_struct {

    // コンテキストスイッチ時にスタックポインタ、インストラクションポインタなどを保存する領域
    struct thread_info      thread_info;
    // プロセスの状態
    volatile long           state;
    // 優先度
    int             prio;
    // プロセスの仮想メモリ空間に関する情報(ページテーブルなど)を記録
    struct mm_struct        *mm;

    // 他にもいろいろとプロセスに関する情報を保持している
    int             static_prio;
    int             normal_prio;
    unsigned int            rt_priority;
    int             exit_state;
    int             exit_code;
    int             exit_signal;
}
システムで実行中のプロセスは、IO待ちなどにより一時的に実行を中断している状態などがあります。
以下はプロセスの状態です。

TASK_RUNNING: 実行中・実行待ちの状態
TASK_STOPPED: 実行中断になった状態
TASK_ZOMBIE: ゾンビプロセス
TASK_UNINTERRUPTABLE: 割り込み不能な待ち状態
TASK_INTERRUPTABLE: 割り込み可能な待ち状態。ハードウェア割り込み、システム資源の解放、シグナルなどの要因により起床
今回取り上げるLoad Averageは上記のうちRUNNINGとUNINTERRUPTABLEの和がLoad Averageとなっています。
アクティブなプロセス数の算出をしている処理。

kernel/sched/loadavg.c
long calc_load_fold_active(struct rq *this_rq, long adjust)
{
    long nr_active, delta = 0;

    nr_active = this_rq->nr_running - adjust;
    nr_active += (long)this_rq->nr_uninterruptible;

    if (nr_active != this_rq->calc_load_active) {
        delta = nr_active - this_rq->calc_load_active;
        this_rq->calc_load_active = nr_active;
    }

    return delta;
}
構造体rqはCPU毎のrunqueueデータ構造を表している
上記で述べた状態のプロセス数などもこの構造体を参照して引っ張り出しています。
※task_struct同様巨大な構造体なので使いそうなのだけ抜き出してます

kernel/sched/sched.h
struct rq {
    /* runqueue lock: */
    raw_spinlock_t      lock;

    unsigned int        nr_running;
    unsigned long       nr_uninterruptible;

    struct task_struct  *curr;
    struct task_struct  *idle;
    struct task_struct  *stop;
PSコマンドで見えるプロセスの状態を表すアルファベットはそれぞれ下記の意味を持つ
この中でもロードアベレージに影響をもたらすのはDとRです。

D Uninterruptible sleep (usually IO)
R Running or runnable (on run queue)
S Interruptible sleep (waiting for an event to complete)
T Stopped, either by a job control signal or because it is being traced.
W paging (not valid since the 2.6.xx kernel)
X dead (should never be seen)
Z Defunct ("zombie") process, terminated but not reaped by its parent.
CPU使用率とLoadAverageの違い
LoadAverageとCPU使用率はすごく混同しがちで現に自分も調べるまで同じニュアンスで使っていた。
実際に調べてみる厳密に違うことが分かった。

LoadAverageは実行状態 + 実行可能状態のプロセス数の平均を示す
それに対しCPU使用率とはプログラムが効率的にCPUを使えているかの値であり
所与の時間に対してどれだけCPUが稼働したかを占める割合を指します。

Load Averageを確認する
Load Averageは以下のコマンドで確認できます

uptime
w
top
sar
uptimeコマンド
システムの稼働時間を見るコマンド
LoadAverageと現在のログインユーザ数も確認できます。

$ uptime
 05:01:16 up 12:52,  2 users,  load average: 0.09, 0.09, 0.04
wコマンド
システムに誰がログインしていて何をしているかを表すコマンド
ログイン夕のほかユーザの端末や実行中のプログラムなんかも見れます。

$ w
 05:00:53 up 12:52,  2 users,  load average: 0.13, 0.10, 0.04
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
ryu      :0       :0               Sat06   ?xdm?   4:59   0.05s /usr/lib/gdm3/gdm-x-session --run-script env GNOME
ryu      pts/1    192.168.1.117    04:59    1.00s  0.10s  0.06s w
topコマンド
システムの様々な統計を同時に表示し変化を監視することができるコマンド
topコマンドについては下記記事が詳細を分かりやすく説明してくれてるのでおススメです。
topコマンドの使い方

$ top
top - 04:59:28 up 12:50,  2 users,  load average: 0.43, 0.12, 0.04
Tasks: 250 total,   2 running, 248 sleeping,   0 stopped,   0 zombie
%Cpu(s): 10.0 us, 22.0 sy,  0.0 ni, 30.0 id, 18.0 wa,  0.0 hi, 20.0 si,  0.0 st
MiB Mem :   1969.9 total,    265.8 free,    962.5 used,    741.6 buff/cache
MiB Swap:   1425.6 total,   1416.0 free,      9.6 used.    804.2 avail Mem

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
  2391 ryu       20   0 2770156 164156  67796 R  31.8   8.1   0:34.09 gnome-shell
 12645 ryu       20   0   35784   3944   3212 R   4.5   0.2   0:00.08 top
sarコマンド
sysstatパッケージに含まれている、システムの統計情報を取得するコマンド
ロードアベレージ以外にも多彩な情報を取得できる。
sarコマンドについては下記記事が詳細を分かりやすく説明してくれてるのでおススメです。
Linux - sarコマンドについて

$ sar -q -s 21:00:00
21時00分01秒   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15
21時05分01秒         1       203      0.19      0.20      0.16
21時10分01秒         0       203      0.21      0.18      0.17
21時15分01秒         2       209      0.11      0.18      0.17
21時20分01秒         1       208      0.38      0.20      0.18
21時25分01秒         0       210      0.10      0.20      0.18
平均値:          1       207      0.20      0.19      0.17
簡単に見方を書くと以下のようになっている。

runq-sz CPUを実行する為のメモリー内で待機中のカーネルスレッド数。通常、この値は 2 未満になる。
plist-sz プロセスリストのプロセスとスレッド数
ldavg-1 過去1分間のロードアベレージ
ldavg-5 過去5分間のロードアベレージ
ldavg-15 過去15分間のロードアベレージ
算出方法
Active なプロセス数は calc_load_tasks 変数で管理されていて、activeなプロセスを算出している。
avenrun[] に保持されています。これは要素数 3 の配列で、それぞれ 1, 5, 15分間の平均値を格納しています。

kernel/sched/loadavg.c
void calc_global_load(unsigned long ticks)
{
    unsigned long sample_window;
    long active, delta;

    sample_window = READ_ONCE(calc_load_update);
    if (time_before(jiffies, sample_window + 10))
        return;

    delta = calc_load_nohz_fold();
    if (delta)
        atomic_long_add(delta, &calc_load_tasks);

    // 現在アクティブなプロセス数を算出
    active = atomic_long_read(&calc_load_tasks);
    active = active > 0 ? active * FIXED_1 : 0;

    // それぞれ 1, 5, 15分間の平均値を格納
    // 実際の計算はcalc_load 関数内で行っている
    avenrun[0] = calc_load(avenrun[0], EXP_1, active);
    avenrun[1] = calc_load(avenrun[1], EXP_5, active);
    avenrun[2] = calc_load(avenrun[2], EXP_15, active);

    WRITE_ONCE(calc_load_update, sample_window + LOAD_FREQ);

    calc_global_nohz();
}

/*
* @param load 前回の Load Average
* @param exp 平均をとる期間ごとに定められた定数
* @param active 現在の Active なプロセス数
*/
static unsigned long
calc_load(unsigned long load, unsigned long exp, unsigned long active)
{
    load *= exp;
    load += active * (FIXED_1 - exp);
    return load >> FSHIFT;
}
余談① : カーネルスレッドとは
カーネルの補助的な処理を行うもの。特定のカーネル関数を実行したりする。
カーネルスレッドを生成する際はexecはしていない。ユーザーモードにならずカーネルスレッドとして生成される。
代表的ななものとして、プロセスIDが1のすべての親となるinitがある。

大きな特徴としてカーネルスレッドはカーネルモードのみでしか実行されない点。
(通常のプロセスはカーネルモードとユーザモードが交互に入れ替わる)

以下はカーネルスレッド生成の実装部分。

arch/x86/kernel/process.c
int kernel_thread(int (*fn)(void *), void *arg, unsigned long flags)
{
        struct pt_regs regs;

        memset(&regs, 0, sizeof(regs));

        regs.si = (unsigned long) fn;
        regs.di = (unsigned long) arg;

#ifdef CONFIG_X86_32
        regs.ds = __USER_DS;
        regs.es = __USER_DS;
        regs.fs = __KERNEL_PERCPU;
        regs.gs = __KERNEL_STACK_CANARY;
#else
        regs.ss = __KERNEL_DS;
#endif

        regs.orig_ax = -1;
        regs.ip = (unsigned long) kernel_thread_helper;
        regs.cs = __KERNEL_CS | get_kernel_rpl();
        regs.flags = X86_EFLAGS_IF | X86_EFLAGS_BIT1;

        /* Ok, create the new process.. */
        return do_fork(flags | CLONE_VM | CLONE_UNTRACED, 0, &regs, 0, NULL, NULL);
}
余談②：スピンロックについて
何度か出てくる用語スピンロックについて。

マルチプロセッサ環境において，各CPUが同じリソースに同時アクセスする際に用いられる排他制御の仕組み。
1つのリソースに対して1つのロック用の変数をメモリー上に展開します。ロック変数を取得できたCPUだけがリソースへアクセスできる。

スレッドが短時間だけブロックされるなら、スピンロックは効率的
ただし，他のCPUがアクセス中は他のCPUは処理が実行できずにwait状態になります。故にEBUSYが頻繁に発生すると処理効率が悪くなるためリソース管理も別途必要になる(細かく分けるなど)。
ロックの同時アクセスの可能性を考慮するのは難しい。

参考：https://ja.wikipedia.org/wiki/%E3%82%B9%E3%83%94%E3%83%B3%E3%83%AD%E3%83%83%E3%82%AF

感想
サーバ側の負荷情報が実際どのように算出されれているのかの実装が分かった。
プロセスの状態だったりは意識したりすることが少ないのでこれからも読み進めていきたい。

まとめ(的なの)
ロードアベレージは、CPUの負荷を表しているわけではなく、
以下の状態のプロセスの数を集計したものです。

①実行中または実行待ちのプロセス
psコマンドで表示した場合に"R"となっているプロセス

②割り込み不可でスリープしているプロセス
psコマンドで表示した場合に"D"となっているプロセス
スリープ状態のため、システムへの負荷はありません。

何で割り込み負荷プロセスもLoadAverage？って思った方は下記を読むことをおススメします。
Linuxが何故このような実装を行ったのか歴史的経緯を説明してくれている記事です。
Linux Load Averages: Solving the Mystery