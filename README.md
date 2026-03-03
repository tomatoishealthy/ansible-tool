# ansible-tool

批量为集群节点配置 SSH 免密互信的 Ansible 工具。

适用场景：Hadoop / HBase / 大数据集群初始化时，需要让所有节点之间 `hadoop` 用户可以互相免密登录。

---

## 原理

1. 在所有节点上创建 `hadoop` 用户并生成 SSH 密钥对
2. 将各节点的公钥（`id_rsa.pub`）汇总到本机 `/tmp/`
3. 合并成统一的 `authorized_keys` 文件
4. 分发到所有节点的 `/home/hadoop/.ssh/authorized_keys`
5. 设置正确的文件权限（`0600`）
6. 清理临时文件

执行完成后，所有节点之间 `hadoop` 用户可以互相免密 SSH 登录。

---

## 依赖

- Ansible（控制机上安装即可）
- 控制机到所有目标节点已有 SSH 访问权限（root 或有 sudo 权限的用户）

---

## 使用方法

**1. 编辑 `host` 文件，填入需要互信的节点**

```ini
# host 组名默认为 ssh
[ssh]
192.168.1.101
192.168.1.102
192.168.1.103
```

**2. 执行 Playbook**

```bash
ansible-playbook ssh.yml -i host
```

**3. 验证**

```bash
# 在任意节点上，以 hadoop 用户 SSH 到其他节点，应无需密码
ssh hadoop@192.168.1.102
```

---

## 文件说明

| 文件 | 说明 |
|------|------|
| `ssh.yml` | 主 Playbook，执行 SSH 互信配置 |
| `host` | Ansible Inventory，填写目标节点列表 |

---

## 注意事项

- Playbook 使用 `become: yes`，执行账号需要有 sudo 权限
- 会在所有节点上**创建** `hadoop` 用户（如已存在则跳过）
- 执行过程中会在控制机 `/tmp/` 下临时存放公钥文件，执行完后自动清理
