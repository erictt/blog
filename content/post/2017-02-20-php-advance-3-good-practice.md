---
layout: post
title: 好的编程习惯 - PHP进阶 (3)
categories: [编程学习]
tags: [PHP]
date: 2017-02-20
description: PHP 好的编程习惯
keywords: Modern PHP, PHP, Note
---

### 净化、验证、转义输入输出

#### 需要注意的几个外部数据源

* $_GET
* $_POST
* $_REQUEST
* $_COOKIE
* $argv
* php://stdin
* php://input
* file_get_contents()
* 远程数据库
* 远程 APIs
* 远程客户端获取的数据

#### 净化输入

* HTML
  * 在接触存储层之前，使用 htmlentities() 来净化 HTML 的特殊字符:

            <?php $input = '<p><script>alert("You won the Nigerian lottery!");</script></p>'; 
            echo htmlentities($input, ENT_QUOTES, 'UTF-8');

  * 避免使用 preg_replace(), preg_replace_all() 和 preg_replace_callback() 等正则函数在HTML代码里。
* SQL 查询
  * 使用 PDO prepared statement 的形式，避免SQL注入等问题。
* 用户信息收集
  * 使用 filter_var() 和 filter_input() 来净化用户信息数据，如 emails, URL-encoded strings, integers, floats, HTML characters, URLs 等，例：

            <?php $email = 'john@example.com'; 
            $emailSafe = filter_var($email, FILTER_SANITIZE_EMAIL);

#### 验证数据

* 可以使用内置函数 filter_var() 来做简单的验证：

        <?php 
        $input = 'john@example.com'; 
        $isEmail = filter_var($input, FILTER_VALIDATE_EMAIL); 
        if ($isEmail !== false) {
            echo "Success"; 
        } else {
            echo "Fail"; 
        }

* 几个推荐的扩展库

  * [aura/filter](https://packagist.org/packages/aura/filter)
  * [respect/validation](https://packagist.org/packages/respect/validation)
  * [symfony/validator](https://packagist.org/packages/symfony/validator)

#### 转义输出

* 使用 htmlentities() ：

        <?php 
        $output = '<p><script>alert("NSA backdoor installed");</script>'; 
        echo htmlentities($output, ENT_QUOTES, 'UTF-8');

* 使用模板引擎，比如：

    [twig/twig](https://packagist.org/packages/twig/twig)

### 密码

#### 几个重要的准则

* 不要记录用户的密码
* 不要过多限制用户的密码，仅推荐限制长度即可
* 不要通过邮件发送用户密码
* 使用 bcrypt 函数单向加密用户密码，个人推荐 [phpass](http://www.openwall.com/phpass/) 第三方扩展

* 用户注册和登录示例

  * 注册时使用 password_hash() 来加密用户密码, 源代码: [codeguy/modern-php](https://github.com/codeguy/modern-php/blob/master/05-good-practices/passwords/register.php)
  * 注：CRYPT_BLOWFISH 加密方式的salt的组成依次是：
    * ("$2a$", "$2x$" 或者 "$2y$") + "xx"(04<xx<31) + "$" + "22个字符"("./0-9A-Za-z")，
    * 例如："$2a$07$usesomesillystringfor"
    * PHP 5.3.7 之后应使用"$2y$"作为前缀

                <?php
                try {
                    // Validate email
                    $email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
                    if (!$email) {
                        throw new Exception('Invalid email');
                    }
                    
                    // Validate password
                    $password = filter_input(INPUT_POST, 'password');
                    if (!$password || mb_strlen($password) < 8) {
                        throw new Exception('Password must contain 8+ characters');
                    }
                    
                    // Create password hash
                    $passwordHash = password_hash(
                       $password,
                       CRYPT_BLOWFISH, // 加密方式
                       ['cost' => 12] // 递归次数
                    );
                    if ($passwordHash === false) {
                        throw new Exception('Password hash failed');
                    }
                    
                    // Create user account (THIS IS PSUEDO-CODE)
                    // $user = new User();
                    // $user->email = $email;
                    // $user->password_hash = $passwordHash;
                    // $user->save();
                    // Redirect to login page
                    
                    header('HTTP/1.1 302 Redirect');
                    header('Location: /login.php');
                } catch (Exception $e) {
                    // Report error
                    header('HTTP/1.1 400 Bad request');
                    echo $e->getMessage();
                }

  * 登录，不要忘了 rehash password ，源代码: [codeguy/modern-php](https://github.com/codeguy/modern-php/blob/master/05-good-practices/passwords/login.php)

            <?php
            session_start();
            try {
                // Get email address from request body
                $email = filter_input(INPUT_POST, 'email');
                
                // Get password from request body
                $password = filter_input(INPUT_POST, 'password');
                
                // Find account with email address (THIS IS PSUEDO-CODE)
                $user = User::findByEmail($email);
                
                // Verify password with account password hash
                if (password_verify($password, $user->password_hash) === false) {
                    throw new Exception('Invalid password');
                }
                
                // Re-hash password if necessary (see note below)
                $currentHashAlgorithm = CRYPT_BLOWFISH;
                $currentHashOptions = array('cost' => 15);
                $passwordNeedsRehash = password_needs_rehash(
                    $user->password_hash,
                    $currentHashAlgorithm,
                    $currentHashOptions
                );
                if ($passwordNeedsRehash === true) {
                
                    // Save new password hash (THIS IS PSUEDO-CODE)
                    $user->password_hash = password_hash(
                        $password,
                        $currentHashAlgorithm,
                        $currentHashOptions
                    );
                    $user->save();
                }
                
                // Save login status to session
                $_SESSION['user_logged_in'] = 'yes';
                $_SESSION['user_email'] = $email;
                
                // Redirect to profile page
                header('HTTP/1.1 302 Redirect');
                header('Location: /user-profile.php');
            } catch (Exception $e) {
                header('HTTP/1.1 401 Unauthorized');
                echo $e->getMessage();
            }

### 流

* 在源与目的地之间传输数据。源和目的地可以是文件、内存、命令行、标准输入输入等。

#### 封装器

* 每个流都有协议和目标，格式为`<scheme>://<target>`，协议<scheme>即为封装器，目标<target>为数据源。常用的封装器有：
  * `file://`. 我们常用的函数如 `file_get_contents()`, `fopen()`, `fwrite()`, `fclose()` 都是流的封装器。
  * `php://`. 标准输入输出的封装器。比如：`php://stdin`, `php://stdout`, `php://memory`, `php://temp`(类似`php://memory`, 只是这个写入的是临时文件)。

#### 上下文

* 用于定义流的行为。比如使用`stream_context_create()`可以让`file_get_contents()`发送 POST 请求：

        <?php
        $requestBody = '{"username":"josh"}'; 
        $context = stream_context_create(array(
            'http' => array( 
                'method' => 'POST', 
                    'header' => "Content-Type: application/json;charset=utf-8;\r\n" . 
                                "Content-Length: " . mb_strlen($requestBody), 
                    'content' => $requestBody
               )
        )); 
        $response = file_get_contents('https://my-api.com/users', false, $context);

#### 过滤器

* 在传输的过程中进行过滤、转换、增加、删除操作。
* 简单的例子，将传输过来的数据全都大写化：

        <?php 
        $handle = fopen('data.txt', 'rb'); 
        stream_filter_append($handle, 'string.toupper'); 
        while(feof($handle) !== true) { 
            echo fgets($handle); // <-- Outputs all uppercase characters 
        } 
        fclose($handle);
* 另一个例子：

        <?php 
        $handle = fopen('php://filter/read=string.toupper/resource=data.txt', 'rb');
        while(feof($handle) !== true) { 
            echo fgets($handle); // <-- Outputs all uppercase characters 
        } 
        fclose($handle);

  * 过滤器写在`php://`的流封装器里，其标准格式为：`filter/read=<filter_name>/resource=<scheme>://<target>`

* 例子3, 使用bzip分开压缩过去30天的日志文件，每天一个压缩包:

        <?php
        $dateStart = new \DateTime();
        $dateInterval = \DateInterval::createFromDateString('-1 day');
        $datePeriod = new \DatePeriod($dateStart, $dateInterval, 30);
        foreach ($datePeriod as $date) {
            $file = 'sftp://USER:PASS@rsync.net/' . $date->format('Y-m-d') . '.log.bz2';
            if (file_exists($file)) {
                $handle = fopen($file, 'rb');
                stream_filter_append($handle, 'bzip2.decompress');
                while (feof($handle) !== true) {
                    $line = fgets($handle);
                    if (strpos($line, 'www.example.com') !== false) {
                        fwrite(STDOUT, $line);
                   }
                }
                fclose($handle);
            }
        }

* 自定义过滤器。需完成以下三点：(参考实例：[DirtyWordsFilter.php](https://github.com/codeguy/modern-php/blob/master/05-good-practices/streams/DirtyWordsFilter.php))
  * 继承内置类`php_user_filter`
  * 实现`filter()`, `onCreate()`, `onClose()`三个方法
  * 使用`stream_filter_register()`注册自定义的过滤器

### 错误和异常处理

#### 异常处理

* 设置全局性的异常捕获来保证程序中未捕获的异常最后都会被记录下来。

        <?php // Register your exception handler 
        set_exception_handler(function (Exception $e) { 
            // Handle and log exception 
        }); 
       
        // Your code goes here...
    
        // Restore previous exception handler 
        restore_exception_handler();

* 在开发环境中使用异常捕获以及日志来显示所有的DEBUG信息，在生产环境使用用户友好的提示信息。

#### 错误

* 四个准则

  * 打开错误报告
  * 在测试环境中显示错误信息
  * 不在生产环境中显示错误信息
  * 无论生产还是开发都应该在日志中记录错误

* 生产环境`php.ini`示例：

        ; DO NOT display errors 
        display_startup_errors = Off    
        display_errors = Off

        ; Report all errors EXCEPT notices 
        error_reporting = E_ALL & ~E_NOTICE

        ; Turn on error logging 
        log_errors = On

* 开发环境示例：

        ; Display errors 
        display_startup_errors = On 
        display_errors = On

        ; Report all errors 
        error_reporting = -1

        ; Turn on error logging 
        log_errors = On

#### 错误处理

* 等同于异常处理，错误也是可以被捕获的。如果只是临时使用错误捕获，一定不要忘了在自己的代码完成后恢复：

        <?php // Register error handler 
        set_error_handler(function ($errno, $errstr, $errfile, $errline) { 
            if (!(error_reporting() & $errno)) { 
                // Error is not specified in the error_reporting 
                // setting, so we ignore it.
                return; 
            }
            throw new ErrorException($errstr, $errno, 0, $errfile, $errline);
        });

        // Your code goes here...

        // Restore previous error handler 
        restore_error_handler();

#### 第三方扩展推荐

* [filp/whoops](https://github.com/filp/whoops) 是一个错误处理框架，提供一个优雅的错误界面帮助DEBUG。目前很多主流的框架比如Laraval 5, CakePHP 3等都已经内置了。
* [Seldaek/monolog](https://github.com/Seldaek/monolog) 记录日志的插件
