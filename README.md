<p align="center">
 <img src="https://user-images.githubusercontent.com/796136/50286124-6f7f3780-046f-11e9-9f45-e8fedd4f786d.png" height="75px" alt="RoadRunner">
</p>
<p align="center">
 <a href="https://packagist.org/packages/spiral/roadrunner"><img src="https://poser.pugx.org/spiral/roadrunner/version"></a>
	<a href="https://pkg.go.dev/github.com/spiral/roadrunner?tab=doc"><img src="https://godoc.org/github.com/spiral/roadrunner?status.svg"></a>
	<a href="https://github.com/spiral/roadrunner/actions"><img src="https://github.com/spiral/roadrunner/workflows/CI/badge.svg" alt=""></a>
	<a href="https://goreportcard.com/report/github.com/spiral/roadrunner"><img src="https://goreportcard.com/badge/github.com/spiral/roadrunner"></a>
	<a href="https://scrutinizer-ci.com/g/spiral/roadrunner/?branch=master"><img src="https://scrutinizer-ci.com/g/spiral/roadrunner/badges/quality-score.png"></a>
	<a href="https://codecov.io/gh/spiral/roadrunner/"><img src="https://codecov.io/gh/spiral/roadrunner/branch/master/graph/badge.svg"></a>
	<a href="https://lgtm.com/projects/g/spiral/roadrunner/alerts/"><img alt="Total alerts" src="https://img.shields.io/lgtm/alerts/g/spiral/roadrunner.svg?logo=lgtm&logoWidth=18"/></a>
	<a href="https://discord.gg/TFeEmCs"><img src="https://img.shields.io/badge/discord-chat-magenta.svg"></a>
	<a href="https://packagist.org/packages/spiral/roadrunner"><img src="https://img.shields.io/packagist/dd/spiral/roadrunner?style=flat-square"></a>
</p>

RoadRunner is an open-source (MIT licensed) high-performance PHP application server, load balancer, and process manager.
It supports running as a service with the ability to extend its functionality on a per-project basis.

RoadRunner includes PSR-7/PSR-17 compatible HTTP and HTTP/2 server and can be used to replace classic Nginx+FPM setup
with much greater performance and flexibility.

<p align="center">
	<a href="https://roadrunner.dev/"><b>Official Website</b></a> | 
	<a href="https://roadrunner.dev/docs"><b>Documentation</b></a>
</p>

Repository:
--------
This repository contains the codebase used to publish Prometheus metrics from PHP application.
Check [spiral/roadrunner](https://github.com/spiral/roadrunner) to get application server.

Installation:
--------
To install RoadRunner extension:

```bash
$ composer require spiral/roadrunner-http spiral/roadrunner-metrics nyholm/psr7
$ vendor/bin/rr get-binary
```

Enable metrics service in your `.rr.yaml` file:

```yaml
rpc:
   listen: tcp://127.0.0.1:6001

server:
   command: "php worker.php"

http:
   address: "0.0.0.0:8080"

metrics:
   address: "0.0.0.0:8081"
```

Example:
-------
To publish metrics from your application worker:

```php
<?php

use Spiral\Goridge;
use Spiral\RoadRunner;
use Nyholm\Psr7\Factory;

ini_set('display_errors', 'stderr');
include "vendor/autoload.php";

$worker = new RoadRunner\Http\PSR7Worker(
    RoadRunner\Worker::create(),
    new Factory\Psr17Factory(),
    new Factory\Psr17Factory(),
    new Factory\Psr17Factory()
);

$metrics = new RoadRunner\Metrics\Metrics(
    Goridge\RPC\RPC::create(RoadRunner\Environment::fromGlobals()->getRPCAddress())
);

$metrics->declare(
    'test',
    RoadRunner\Metrics\Collector::counter()->withHelp('Test counter')
);

while ($req = $worker->waitRequest()) {
    try {
        $rsp = new \Nyholm\Psr7\Response();
        $rsp->getBody()->write("hello world");

        $metrics->add('test', 1);

        $worker->respond($rsp);
    } catch (\Throwable $e) {
        $worker->getWorker()->error((string)$e);
    }
}
```

<a href="https://spiral.dev/">
<img src="https://user-images.githubusercontent.com/773481/220979012-e67b74b5-3db1-41b7-bdb0-8a042587dedc.jpg" alt="try Spiral Framework" />
</a>

Testing:
--------
This codebase is automatically tested via host repository - [spiral/roadrunner](https://github.com/spiral/roadrunner).

License:
--------
The MIT License (MIT). Please see [`LICENSE`](./LICENSE) for more information. Maintained
by [Spiral Scout](https://spiralscout.com).
