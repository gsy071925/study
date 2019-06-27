# storm下中文乱码
## 背景
storm提交topo，工程中读取json文件，文件中含有中文参数，在旧环境中吴乱码问题，怀疑迁移到D1新storm集群中导致
重新打包topo，在程序中打印字符集编码输出如下：
file.encoding : ANSI_X3.4-1968
file.encoding.pkg : sun.io
sun.io.unicode.encoding : UnicodeLittle
sun.jnu.encoding : ANSI_X3.4-1968

## 解决
在工程中，修改Config，增加TOPOLOGY_WORKER_CHILDOPTS配置属性：
import org.apache.storm.Config;
Config conf = new Config();
conf.put(Config.TOPOLOGY_WORKER_CHILDOPTS, "-Dfile.encoding=UTF-8");

提交后个worker的打印的字符集编码如下
file.encoding : UTF-8
file.encoding.pkg : sun.io
sun.io.unicode.encoding : UnicodeLittle
sun.jnu.encoding : ANSI_X3.4-1968

正在跑的spot能正常读取json文件