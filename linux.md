# 查找复制30内变更的文件
find /attachment/ -type f -mtime -30|awk '{print $1}'|xargs -i cp {} /attachment/ece_l30/