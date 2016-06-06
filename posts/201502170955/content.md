title: Homestead 2.0.9 的一个坑
date: 2015-02-17 09:55
---

整个下午都在调戏 [Laravel 5](http://laravel.com/docs/5.0) 和 [Homestead](http://laravel.com/docs/5.0/homestead)，然后掉坑里了……记录一下

Homestead.yaml 配置文件中，folders列，如果要设置为当前路径，不能够这样：

```yaml
    folders:
        - map: .
          to: /home/vagrant/your-dir
```

而是要这样：

Homestead.yaml 配置文件中，folders列，如果要设置为当前路径，不能够这样：

```yaml
    folders:
        - map: ./
          to: /home/vagrant/your-dir
```

就因为这尼玛的少了个斜杠 `/`， `/vagrant` 一直mount不了……
