# log4j 配置解析

```plantuml
class RollingFileAppender {
   - long maxFileSize = 10*1024*1024
   - int  maxBackupIndex = 1
   - long nextRollover = 0
}

FileAppender <|-- RollingFileAppender

```