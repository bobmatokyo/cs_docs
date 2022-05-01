topコマンドの使い方


得られる情報
システム全体の負荷
プロセス, CPU, メモリ, スワップの統計情報
使い方
$ top          # CPU使用率順にソート
$ top -a       # メモリ使用率順にソート
$ top -p [PID] # 特定のプロセスを監視
$ top -d1      # 1秒ごとに更新
操作方法
Shift+o: 表示された特定のキーを押してEnterすると任意の列でソートできる
Shift+p: CPU使用率順にソート
Shift+m: メモリ使用率順にソート
ヘッダーの見方
load average
top - 08:42:47 up 2min,  2 users,  load average: 2.76, 0.76, 0.27
#     現在時間 サーバーの  ログイン                 1分   5分   15分
#              稼働時間   ユーザー数               間の単位時間あたりの待ちタスク数
この行はuptimeと同じものが表示される。

Tasks
Tasks: 110 total,   7 running, 103 sleeping,   0 stopped,   0 zombie
#      合計タスク数   稼働中       待機中        停止タスク    ゾンビタスク
各状態のタスクの数を見ることができる。

CPU
Cpu(s): 77.1%us,  8.4%sy,  0.0%ni,  0.1%id, 14.3%wa,  0.0%hi,  0.2%si,  0.0%st
#       user      system    nice     idle   I/O wait hardware  software  steal
#                                                    interrupt interrupt
ni: niceで実行優先度を変更したプロセスがユーザモードでCPUを消費した時間
st: OS仮想化利用時にほかの仮想CPUの計算で待たされた時間
sarのようにCPUの利用時間の割合をプロセスの種類ごとに見ることができる。
1を押すとCPUごとのデータが見れる。

Memory / Swap
Mem:  15144564k total,  1178112k used, 13966452k free,    28300k buffers
Swap:        0k total,        0k used,        0k free,   289928k cached
buffers: mallocなど、バッファとして利用されているメモリ量
cached: キャッシュとして利用されているメモリ量 (ファイルシステムのキャッシュ)
物理メモリとスワップ領域の使用状況が表示される。 free でも見ることができる。

プロセス一覧


PR	NI	VIRT	RES	SHR	S	%CPU	%MEM	TIME+
優先度	相対優先度	仮想メモリ	物理メモリ	共有メモリ	状態	CPU使用率	メモリ使用率	実行時間
NI: Nice value。相対優先度。0が基準で、負だと優先度が高く、正だと優先度が低い。
VIRT: Virtual Image。確保された仮想メモリ全て。スワップしたメモリ使用量を含む。
RES: Resident size。 スワップしていない、使用した物理メモリのサイズ。
SHR: Shared Mem size。 他のプロセスと共有される可能性のあるメモリのサイズ。
S: Process Status。 以下のいずれの状態であるかを示している。
D: 割り込み不能
R: 実行中
S: スリープ状態
T: 停止中
Z: ゾンビプロセス
まとめ
topを使うと、uptime, sar, freeの結果を見つつ各プロセスの状態を監視することができる。