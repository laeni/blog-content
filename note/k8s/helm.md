# Helm



## 常用内置函数

1. trunc - 截断超过阈值的字符串

   ```go
   // 超过 63 字符的部分被截断
   {{- .Values.fullnameOverride | trunc 63 -}}
   ```

2. trimSuffix - 去掉末尾的指定字符串

   ```go
   {{- "abc-xxx" | trimSuffix "-xxx" -}}
   ```

3. printf - 字符串格式化函数（`fmt.Sprintf`）

4. indent

5. nindent

6. quote

7. default