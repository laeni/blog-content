# 常见脚本

- 定期删除日志

  - remove-logs-app.sh

    ```bash
    #!/bin/bash
    
    ## 此脚本通过 cron 任务定期执行，详情可通过`crontab -i`查看
    
    log_dir=$1
    
    # 压缩日志文件
    find ${log_dir} \
      -type f \
      \( -name "*.log" -o -name "*.log.*" \) `# 根据名称过滤` \
      ! -name '*.gz' `# 根据名称排除` \
      -mtime +2 `# 查找 2 天前的` \
      -writable `# 仅查找可写的文件` \
      -exec gzip {} \;
    
    # 删除 180 天以前的所有文件
    find ${log_dir} -name "*.gz" -type f -mtime +180 -writable -exec rm {} \;
    # 删除空目录
    find ${log_dir} -type d -empty -delete
    ```


  - remove-logs-tomcat.sh

    ```bash
    #!/bin/bash
    
    ## 此脚本通过 cron 任务定期执行，详情可通过`crontab -i`查看
    
    tomcat_home=$1
    log_dir=$tomcat_home/logs
    
    echo '' > $log_dir/catalina.out
    
    # 压缩日志文件
    find ${log_dir} \
      -type f \
      \( -name "*.log" -o -name "*.log.*" -o -name "*.txt" \) `# 根据名称过滤` \
      ! -name '*.gz' `# 根据名称排除` \
      -mtime +2 `# 查找 2 天前的` \
      -writable `# 仅查找可写的文件` \
      -exec gzip {} \;
    
    # 删除 60 天以前的所有文件
    find ${log_dir} \( -name "*.log" -o -name "*.txt" -o -name "*.gz" \) -type f -mtime +60 -writable -exec rm {} \;
    # 删除空目录
    find ${log_dir} -type d -empty -delete
    ```

  > Crontab -e:
  >
  > ```
  > 15 0 * * * /opt/appl/remove-logs-app.sh /opt/appl/spring-cloud/logs
  > 16 0 * * * /opt/appl/remove-logs-app.sh /opt/appl/tomcat/logs
  > 17 0 * * * /opt/appl/remove-logs-tomcat.sh /opt/appl/tomcat/apache-tomcat-shield-acl
  > 18 0 * * * /opt/appl/remove-logs-tomcat.sh /opt/appl/tomcat/apache-tomcat-shield-report
  > 19 0 * * * /opt/appl/remove-logs-tomcat.sh /opt/appl/tomcat/apache-tomcat-shield-event
  > ```

