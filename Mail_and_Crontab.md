# Mail and Crontab

## Crontab

`/etc/crontab`:
```bash
MAILTO="2199243629@qq.com,root,aloha"
# run-parts
# 分钟 小时 几号 月份 星期几
# 每个小时 00 分钟
00  *  *  *  * root run-parts /etc/cron.hourly
# 每天 12:00
00 12  *  *  * root run-parts /etc/cron.daily
# 每个星期的星期天 12:00
00 12  *  *  7 root run-parts /etc/cron.weekly
# 每个月 1 号 12:00
00 12  1  *  * root run-parts /etc/cron.monthly

*/1 * * * * aloha echo 'hello'
```

## Mail

```bash
mail -s "hello world" 2199243629@qq.com
```
