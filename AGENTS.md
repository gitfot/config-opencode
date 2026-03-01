Always respond in Chinese-simplified

## WSL Environment Detection & Build Commands

### Environment Detection

**判断规则**: 使用 `pwd` 检查当前路径是否以 `/mnt/` 开头。
- `/mnt/` 开头 → WSL 环境（Windows 磁盘映射），编译通过 `powershell.exe` 调用 Windows 工具链
- 非 `/mnt/` 开头 → 原生 Linux 环境，直接使用本地工具链编译

### WSL Path Conversion

| WSL Path | Windows Path |
|----------|-------------|
| `/mnt/d/...` | `D:\...` |

转换逻辑: `/mnt/{drive_letter}/remaining/path` → `{DRIVE_LETTER}:\remaining\path`（路径分隔符 `/` → `\`）

---

### WSL Java Build (via PowerShell)

**工具链位置**（Windows 路径）：
- JDK 目录: `D:\wanwan\project\common\java_tools\`
  - 可用版本: `jdk1.8.0_261`, `jdk-11.0.17`, `jdk-13.0.1`, `jdk-17.0.8`
- Maven: `D:\wanwan\project\common\java_tools\apache-maven-3.9.0\bin\mvn.cmd`

**命令模板**（`{WIN_PROJECT_DIR}` 为项目路径的 Windows 格式, `{JDK_DIR}` 为对应版本 JDK 目录名）:

```bash
# 完整编译
powershell.exe -Command "\$env:JAVA_HOME='D:\wanwan\project\common\java_tools\{JDK_DIR}'; \$env:PATH=\"\$env:JAVA_HOME\bin;\$env:PATH\"; cd '{WIN_PROJECT_DIR}'; D:\wanwan\project\common\java_tools\apache-maven-3.9.0\bin\mvn.cmd clean compile -DskipTests"

# 单模块编译（{MODULE_NAME} 替换为模块名）
powershell.exe -Command "\$env:JAVA_HOME='D:\wanwan\project\common\java_tools\{JDK_DIR}'; \$env:PATH=\"\$env:JAVA_HOME\bin;\$env:PATH\"; cd '{WIN_PROJECT_DIR}'; D:\wanwan\project\common\java_tools\apache-maven-3.9.0\bin\mvn.cmd clean compile -pl {MODULE_NAME} -am -DskipTests"

# 运行测试
powershell.exe -Command "\$env:JAVA_HOME='D:\wanwan\project\common\java_tools\{JDK_DIR}'; \$env:PATH=\"\$env:JAVA_HOME\bin;\$env:PATH\"; cd '{WIN_PROJECT_DIR}'; D:\wanwan\project\common\java_tools\apache-maven-3.9.0\bin\mvn.cmd test -Dtest={TEST_CLASS}"
```

### WSL Python (via PowerShell)

**工具链位置**（Windows 路径）：
- Python 目录: `C:\Users\for97\AppData\Local\Programs\Python`
  - 可用版本: `Python312`
  - 可执行文件: 各版本目录下的 `python.exe`

**命令模板**（`{WIN_PROJECT_DIR}` 为项目路径的 Windows 格式, `{PYTHON_DIR}` 为对应版本目录名，如 `Python312`）:

```bash
# 运行 Python 脚本
powershell.exe -Command "\$env:PYTHONIOENCODING='utf-8'; cd '{WIN_PROJECT_DIR}'; C:\Users\for97\AppData\Local\Programs\Python\{PYTHON_DIR}\python.exe {SCRIPT_NAME}"

# 安装依赖（pip install）
powershell.exe -Command "\$env:PYTHONIOENCODING='utf-8'; cd '{WIN_PROJECT_DIR}'; C:\Users\for97\AppData\Local\Programs\Python\{PYTHON_DIR}\python.exe -m pip install -r requirements.txt"

# 运行模块
powershell.exe -Command "\$env:PYTHONIOENCODING='utf-8'; cd '{WIN_PROJECT_DIR}'; C:\Users\for97\AppData\Local\Programs\Python\{PYTHON_DIR}\python.exe -m {MODULE_NAME}"

# 运行测试（pytest）
powershell.exe -Command "\$env:PYTHONIOENCODING='utf-8'; cd '{WIN_PROJECT_DIR}'; C:\Users\for97\AppData\Local\Programs\Python\{PYTHON_DIR}\python.exe -m pytest {TEST_PATH}"
```

---

### Native Linux Build

非 WSL 环境下，直接使用系统已安装的工具：

**Java/Maven:**
```bash
mvn clean compile -DskipTests
mvn clean package -DskipTests
mvn clean compile -pl {MODULE_NAME} -am -DskipTests
mvn test -Dtest={TEST_CLASS}
```

---
### Important Notes
1. **WSL 环境下禁止直接使用 Linux 工具链编译 `/mnt/` 下的项目**: WSL 访问 `/mnt/` 下的 Windows 文件系统 I/O 极慢，**必须**通过 `powershell.exe` 调用 Windows 原生工具链（pnpm/npm/maven 均适用）
2. **环境判断只用 `pwd`**: 路径以 `/mnt/` 开头 → WSL 环境，走 PowerShell；否则 → 原生 Linux，直接执行
3. **路径中的转义**: PowerShell 命令中 `$` 需转义为 `\$`，内层双引号需转义为 `\"`
4. **编译前先确认 JDK 版本**: 错误的 JDK 版本会导致编译失败，务必先检查 `pom.xml`
