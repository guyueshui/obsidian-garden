---
{"dg-publish":true,"permalink":"/__zettel/202502201432python-paramiko使用小记/","title":202502201432,"tags":["paramiko","python","ssh"],"created":"2025-02-20T14:32:09+08:00"}
---

paramiko是一个python ssh库，提供python api去执行ssh操作，包括但不限于登录远程机器（ssh），执行远程命令，sftp上传下载文件。

```python
# env: python3
import os
import tarfile
import subprocess
import paramiko
import paramiko.util
paramiko.util.log_to_file("paramiko.log")


def notify(*msg, **kwargs):
    print("--- ", *msg, **kwargs)

def tar_dir(dir_path, tar_path):
    """ Tar a directory `dir_path` to a tar file. """
    if tar_path.endswith(".gz") or tar_path.endswith(".tgz"):
        mode = "w:gz"
    elif tar_path.endswith(".bz2"):
        mode = "w:bz2"
    elif tar_path.endswith(".xz"):
        mode = "w:xz"
    else:
        mode = "w"
    with tarfile.open(tar_path, mode) as tar:
        tar.add(dir_path, arcname=os.path.basename(dir_path))

def untar_file(tar_path, target_dir):
    with tarfile.open(tar_path, "r") as tar:
        tar.extractall(target_dir)

def test_tar():
    tar_dir("./", "/home/bingbing.hu/my.tar.gz")
    untar_file("/home/bingbing.hu/my.tar.gz", "/home/bingbing.hu/mytar_gz_extracted")

class CommandRunException(Exception):
    pass


class NodeConf(object):
    HOST = ""
    PORT = 22
    USERNAME = "fuck"
    PASSWORD = "you"

class NewApiDevNode(NodeConf):
    HOST = "1.2.3.4"

class SSHConnection(object):
    def __init__(self, host="", password="", username="bingbing.hu", port=22) -> None:
        self.host = host
        self.password = password
        self.username = username
        self.port = port
        self._client = None      # type: paramiko.SSHClient | None
        self._sftp_client = None # type: paramiko.SFTPClient | None

    def close(self):
        if self._client is not None:
            self._client.close()
        if self._sftp_client is not None:
            self._sftp_client.close()
        self._client = None
        self._sftp_client = None

    def re_init(self, node_cls):
        """ Re-init the node with a NodeConf class. """
        self.host = node_cls.HOST
        self.password = node_cls.PASSWORD
        self.username = node_cls.USERNAME
        self.port = node_cls.PORT
        if self._client is not None:
            self._client.close()
            self._client = None
        self.get_client()

    def get_client(self):
        if self._client is None:
            self._client = ssh = paramiko.SSHClient()
            ssh.load_system_host_keys()
            ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
            ssh.connect(self.host, self.port, self.username, self.password)
        return self._client

    def get_sftp_client(self):
        if self._sftp_client is None:
            self._sftp_client = self.get_client().open_sftp()
        return self._sftp_client

    def excute(self, cmd):
        stdin, stdout, stderr = self.get_client().exec_command(cmd)
        return self.__handle_cmd_output(stdout, stderr, cmd)

    def excute_with_sudo(self, cmd):
        stdin, stdout, stderr = self.get_client().exec_command("sudo -S %s" % cmd)
        stdin.write(self.password + "\n")
        stdin.flush()
        return self.__handle_cmd_output(stdout, stderr, cmd)

    @staticmethod
    def __handle_cmd_output(stdout: paramiko.ChannelFile, stderr: paramiko.ChannelFile, cmd: str) -> int:
        status = stdout.channel.recv_exit_status()
        if status != 0:
            raise CommandRunException(
                "Excute '%s' failed with status %d.\n\n%s"
                % (cmd, status, stderr.read().decode()))
        for line in stdout:
            print(line, end="")
        return status

    def upload(self, local_path, remote_path):
        if not os.path.exists(local_path):
            raise FileNotFoundError("Local file %s not found." % local_path)
        if remote_path[-1] == '/': # treat remote_path as a directory
            self.__ensure_remote_path(remote_path)
            filename = os.path.basename(local_path)
            remote_path += filename
        notify("Uploading %s to %s" % (local_path, remote_path))

        # \r is used to move cursor to the beginning of the line.
        cb = lambda x, y: notify(f"Uploaded {x}/{y}\r", end="", flush=True)
        self.get_sftp_client().put(local_path, remote_path, callback=cb)
        print('') # used to remove the last \r

    def download(self, remote_path, local_path):
        if remote_path[-1] == '/':
            raise ValueError("Remote path should be a file path.")
        local_file = local_path
        if local_path[-1] == '/':
            local_dir = local_path[:-1]
            local_file = os.path.join(local_path, os.path.basename(remote_path))
        else:
            local_dir = os.path.split(local_path)[0]
        if not os.path.exists(local_dir):
            os.makedirs(local_dir)

        sftp = self.get_sftp_client()
        notify("Downloading %s to %s" % (remote_path, local_file))

        # \r is used to move cursor to the beginning of the line.
        cb = lambda x, y: print(f"\rDownloaded {x}/{y}", end="", flush=True)
        sftp.get(remote_path, local_file, callback=cb)

    def __ensure_remote_path(self, remote_path):
        self.excute(f"""
            if [ ! -d {remote_path} ]; then
                mkdir -p {remote_path}
            fi
            """)

def deploy_newapi_dev():
    conn = SSHConnection()
    conn.re_init(NewApiDevNode)

    # code_root = os.path.expanduser("~/code/newapi")
    # print(code_root)

    args = ["/datayes/api_dev/bin", "api_service.tar.gz"]
    # subprocess.check_call(["make", "-C", *args])
    local_file = '/'.join(args)
    conn.upload(local_file, './')
    conn.excute(f"""
    mkdir ttt
    tar -xavf {args[1]} -C ttt
""")
    conn.close()

def test_connection():
    conn = SSHConnection()
    conn.re_init(NewApiDevNode)

    a = conn.excute("pwd;df -h")
    conn.excute_with_sudo("apt list --upgradable | head -n5")
    conn.excute("""
cd /tmp
mkdir -p test
cd test
pwd
touch {a,b,c}
ls -l""")

    conn.upload("main.py", "./areyouok/")
    # conn.excute("./a.out")
    # node.get_sftp_client().put("main.py", "main.py")
    conn.download("./api_etc.tar", "./from_remote.tar")
    conn.close()
    print(NewApiDevNode.PASSWORD)
    print(NewApiDevNode.USERNAME)



if __name__ == "__main__":
    deploy_newapi_dev()

```

closed by https://github.com/guyueshui/dotfiles/tree/master/bin/paramiko

---

更新下wrapper

node.py
```py
# encoding = utf-8
# This file stores the information of remote nodes.

class NodeConf(object):
    HOST = ""
    PORT = 22
    USERNAME = "bingbing.hu"
    PASSWORD = "datayes@123"

    @classmethod
    def get_para(cls):
        return (cls.HOST, cls.PASSWORD, cls.USERNAME, cls.PORT)

    @classmethod
    def get_para_dict(cls):
        return {"host": cls.HOST, "password": cls.PASSWORD,
                "username": cls.USERNAME, "port": cls.PORT }

class NewApiDevNode(NodeConf):
    HOST = "10.24.21.23"

class DbStoreNode(NodeConf):
    HOST = "10.24.21.105"

class MdlDevNode(NodeConf):
    HOST = "10.24.21.40"

class DbAlchemyNodeStg(NodeConf):
    HOST = "10.24.21.25"
    USERNAME = "xufei.li"

class BarDev(NodeConf):
    HOST = "10.24.21.181"
    USERNAME = "xufei.li"

    DEPLOY_DIR = "/home/shang/mdl_bar"

```

ssh_connection.py
```py
# encoding: utf-8

from tool.utils import notify
import os
import time
import paramiko
import paramiko.util
paramiko.util.log_to_file("paramiko.log")


class CommandRunException(Exception):
    pass


class SSHConnection(object):
    def __init__(self, host="", password="", username="bingbing.hu", port=22) -> None:
        self.host = host
        self.password = password
        self.username = username
        self.port = port
        self._client = None      # type: paramiko.SSHClient | None
        self._sftp_client = None # type: paramiko.SFTPClient | None

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("type: %s" % exc_type)
        print("val: %s" % exc_val)
        print("tb: %s" % exc_tb)
        self.close()

    def close(self):
        if self._client is not None:
            self._client.close()
        if self._sftp_client is not None:
            self._sftp_client.close()
        self._client = None
        self._sftp_client = None

    def re_init(self, node_cls):
        """ Re-init the node with a NodeConf class. """
        self.host = node_cls.HOST
        self.password = node_cls.PASSWORD
        self.username = node_cls.USERNAME
        self.port = node_cls.PORT
        if self._client is not None:
            self._client.close()
            self._client = None
        self.get_client()

    def get_client(self):
        if self._client is None:
            self._client = ssh = paramiko.SSHClient()
            ssh.load_system_host_keys()
            ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
            ssh.connect(self.host, self.port, self.username, self.password)
        return self._client

    def get_sftp_client(self):
        if self._sftp_client is None:
            self._sftp_client = self.get_client().open_sftp()
        return self._sftp_client

    def excute(self, cmd):
        stdin, stdout, stderr = self.get_client().exec_command(cmd)
        return self.__handle_cmd_output(stdout, stderr, cmd)

    def excute_with_sudo(self, cmd):
        stdin, stdout, stderr = self.get_client().exec_command("sudo -S %s" % cmd)
        stdin.write(self.password + "\n")
        stdin.flush()
        return self.__handle_cmd_output(stdout, stderr, cmd)

    def __excute_successive_cmds(self, cmd):
        client = self.get_client()
        chan = client.invoke_shell()
        chan.set_combine_stderr(True)
        stdin = chan.makefile_stdin("wb", -1)
        stdout = chan.makefile("r", -1)
        # chan.send("sudo -i\n")
        # chan.sendall(self.password + "\n")
        # time.sleep(.5)
        # stdin.flush()
        stdin.write("ls -al\n")
        stdin.write(f"""
sudo -i
{self.password}
""")
        time.sleep(0.5)
#         chan.send("""
# pwd
# echo $HOME
# echo $USER
# """.encode())
        # chan.sendall("pwdx 15391\n".encode())
        chan.send("exit\n".encode())
        chan.send("./a.out\n".encode())
        chan.send("exit\n".encode())
        while True:
            if chan.recv_ready():
                print(chan.recv(1024).decode())
                continue
            if chan.exit_status_ready():
                exit_status = chan.recv_exit_status()
                break
            if chan.closed or chan.eof_received or not chan.active:
                break
            time.sleep(0.5)
        print("exit_status:", exit_status)

    @staticmethod
    def __handle_cmd_output(stdout: paramiko.ChannelFile, stderr: paramiko.ChannelFile, cmd: str) -> int:
        for line in stdout:
            print(line, end="")
        status = stdout.channel.recv_exit_status()
        if status != 0:
            raise CommandRunException(
                "Excute '%s' failed with status %d.\n\n%s"
                % (cmd, status, stderr.read().decode()))
        return status

    def get_cmd_chain(self, timeout=0):
        return CommandChain(self.get_client(), timeout)

    def upload(self, local_path, remote_path):
        if not os.path.exists(local_path):
            raise FileNotFoundError("Local file %s not found." % local_path)
        if remote_path[-1] == '/': # treat remote_path as a directory
            self.__ensure_remote_path(remote_path)
            filename = os.path.basename(local_path)
            remote_path += filename
        notify("Uploading %s to %s" % (local_path, remote_path))

        # \r is used to move cursor to the beginning of the line.
        cb = lambda x, y: notify(f"Uploaded {x}/{y}\r", end="", flush=True)
        self.get_sftp_client().put(local_path, remote_path, callback=cb)
        print('') # used to remove the last \r

    def download(self, remote_path, local_path):
        if remote_path[-1] == '/':
            raise ValueError("Remote path should be a file path.")
        local_file = local_path
        if local_path[-1] == '/':
            local_dir = local_path[:-1]
            local_file = os.path.join(local_path, os.path.basename(remote_path))
        else:
            local_dir = os.path.split(local_path)[0]
        if not os.path.exists(local_dir):
            os.makedirs(local_dir)

        sftp = self.get_sftp_client()
        notify("Downloading %s to %s" % (remote_path, local_file))

        # \r is used to move cursor to the beginning of the line.
        cb = lambda x, y: print(f"\rDownloaded {x}/{y}", end="", flush=True)
        sftp.get(remote_path, local_file, callback=cb)

    def __ensure_remote_path(self, remote_path):
        self.excute(f"""
            if [ ! -d {remote_path} ]; then
                mkdir -p {remote_path}
            fi
            """)


class CommandChain(object):
    def __init__(self, ssh_client: paramiko.SSHClient, timeout=0):
        assert ssh_client is not None
        self._client = ssh_client
        self._chan = ssh_client.invoke_shell(self.__class__.__name__) # type: paramiko.Channel
        time.sleep(1) # wait the channel to be ready
        # self._chan.setblocking(0)
        # self._chan.set_combine_stderr(True)
        self._stdin = self._chan.makefile_stdin("wb", -1)
        self._exit_status = 0
        self._timeout = timeout

    def over(self):
        time.sleep(0.5) # essential, otherwise the command will be blocked
        # while not self._chan.exit_status_ready():
        #     self.execute("exit")
        self.__handle_output()
        self._stdin.close()
        self._chan.close()
        self._client = None

    def execute(self, one_line_cmd: str):
        one_line_cmd = one_line_cmd.rstrip('\n') + '\n'
        self._chan.send(one_line_cmd.encode())
        return self

    def write_input(self, text: str):
        text = text.rstrip('\n') + '\n'
        self._stdin.write(text.encode())
        return self

    def __handle_output(self):
        chan = self._chan
        # If you don't set timeout, the stdout.read will block,
        # or you can send multiple "exit" to remote node,
        # this will make chan.exit_status_ready returns true.
        if self._timeout > 0:
            chan.settimeout(5)
        stdout = chan.makefile("r", -1)
        try:
            for line in stdout:
                print(line, end="")
        except Exception as e:
            notify("no data in %ss, channel will be closed!" % self._timeout)


def test_connection():
    from tool.node import NewApiDevNode
    conn = SSHConnection()
    conn.re_init(NewApiDevNode)

    conn.excute("pwd;df -h")
    conn.excute_with_sudo("apt list --upgradable | head -n5")
    conn.excute("""
cd /tmp
mkdir -p test
cd test
pwd
touch {a,b,c}
ls -l""")

    conn.upload("main.py", "./areyouok/")
    # conn.excute("./a.out")
    # node.get_sftp_client().put("main.py", "main.py")
    conn.download("./api_etc.tar", "./from_remote.tar")
    conn.close()


def test_cmd_chain():
    from tool.node import NewApiDevNode
    conn = SSHConnection()
    conn.re_init(NewApiDevNode)

    conn.get_cmd_chain().execute("echo $USER")\
        .execute("sudo -i").write_input(conn.password)\
        .execute("echo $USER").execute("apt list --upgradable|head -5")\
        .over()


if __name__ == "__main__":
    # test_connection()
    test_cmd_chain()
```

utils.py
```py
# encoding: utf-8

import os
import tarfile

def notify(*msg, **kwargs):
    print("---", *msg, **kwargs)

def __tar_mode_helper(tar_path: str):
    if tar_path.endswith(".gz") or tar_path.endswith(".tgz"):
        mode = "w:gz"
    elif tar_path.endswith(".bz2"):
        mode = "w:bz2"
    elif tar_path.endswith(".xz"):
        mode = "w:xz"
    else:
        mode = "w"
    return mode

def tar_dir(dir_path: str, tar_path: str, **kwargs):
    """ Tar a directory `dir_path` to a tar file. """
    mode = __tar_mode_helper(tar_path)
    with tarfile.open(tar_path, mode) as tar:
        tar.add(dir_path, arcname=os.path.basename(dir_path), **kwargs)

def tar_files(tar_path: str, *files):
    mode = __tar_mode_helper(tar_path)
    with tarfile.open(tar_path, mode) as tar:
        for fn in filter(lambda x: os.path.exists(x), files):
            tar.add(fn, arcname=os.path.basename(fn))

def untar_file(tar_path, target_dir):
    with tarfile.open(tar_path, "r") as tar:
        tar.extractall(target_dir)

def test_tar():
    tar_dir("./", "/home/bingbing.hu/my.tar.gz")
    untar_file("/home/bingbing.hu/my.tar.gz", "/home/bingbing.hu/mytar_gz_extracted")


if __name__ == "__main__":
    test_tar()
```

示例
```py
from common import *
import os
import tarfile


class CompileTask1604(object):
    """ Compile on ubuntu 16.04. """
    def __init__(self) -> None:
        self.conn = SSHConnection()
        self.conn.re_init(N.NewApiDevNode)

    def destroy(self):
        if self.conn is not None:
            self.conn.close()
        self.conn = None

    @staticmethod
    def filter_src_files(tarinfo: tarfile.TarInfo):
        name = tarinfo.name
        if '.git' in name or ".vscode" in name or ".cache" in name:
            return None
        if name.startswith('.'):
            return None
        if tarinfo.isdir():
            if "3rd" in name or "build" in name or "doc" in name or "obj" in name or "bin" in name:
                return None
            return tarinfo
        if tarinfo.isfile() and (
            name.endswith(".h") or name.endswith(".cpp")
        ):
            return tarinfo
        return None

    def compile_mdl_src(self):
        conn = self.conn

        tar_name = "mdl_src.tar.bz2"
        utils.tar_dir(MDL_SRC, tar_name, filter=self.__class__.filter_src_files)
        conn.upload(tar_name, "./code/")
        conn.excute(f"""
cd code
tar -xavf {tar_name}
rm -f {tar_name}
echo; echo
cd mdl_src/build/make
make -j4
""")
        os.remove(tar_name)

    def compile_dbalchemy(self):
        conn = self.conn

        def _filter(tarinfo: tarfile.TarInfo):
            name = tarinfo.name
            if 'build.sh' in name or "makefile" in name:
                return tarinfo
            return self.__class__.filter_src_files(tarinfo)

        tar_name = "dbalchemy.tar.bz2"
        utils.tar_dir(DBALCHEMY, tar_name, filter=_filter)
        conn.upload(tar_name, "./code/")
        conn.excute(f"""
set -e
cd code
tar -xavf {tar_name}
rm -f {tar_name}
echo; echo
cd dbalchemy/
bash -e build.sh
""")
        os.remove(tar_name)


if __name__ == '__main__':
    task = CompileTask1604()
    task.compile_mdl_src()
    task.destroy()
```