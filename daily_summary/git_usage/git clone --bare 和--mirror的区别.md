# git clone --bare 和--mirror的区别

git clone --bare XXX是得到一个裸仓库，只包含.git文件

git clone --mirror XXX的到一个可以继续跟踪原仓库的裸仓库，只包含.git 文件

--bare:得到的.git文件中不能进行fetch和pull;而--mirror得到的.git文件可以进行fetch和pull,依然可以获取原仓库最近的更新。