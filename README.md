# 基于redis的任务队列服务   

## 功能    
1.可指定任务处理的类和方法 
2.出队备份，防止任务执行过程中意外退出造成数据丢失      
3.支持失败重试策略（延时、次数）
4.除redis、php，无需额外启动任何服务    

## 原理   
1.借助redis数据结构：用一个list做队列结构，一个zset做备份，一个zset做延时重试
2.用setnx模拟读锁，保证并发安全
3.采用“容器模式”，实现各worker对象的单例化
    
# 使用       
1.必须要先引入autoload.php，如下：  

	require_once '../autoload.php';  
	
2.引入相关的RedisQueue类及命名空间，如下：  
	
	use Src\Space\Phper\Task\RedisQueue;  
	
3.push 

    // 此处指定该任务由Test\TestWorker类的fire方法执行
	$job = 'Test\TestWorker@fire';
	$data = array('hello' => 'push');
	$redisQueue->push($job, $data);
	
4.pop&fire    

    $task = $redisQueue->pop();
    $task->fire();

5.可外部注入worker   

	// 一般来说，不需要外部注入worker，程序会通过new的方式自动创建worker；
	// 但是，对于CodeIgniter等不支持命名空间的框架，必须先自行往WorkerContainer注入Worker对象
	// 如下，取worker容器类，通过外部注入worker
	$container = WorkerContainer::getInstance();
    $task = $redisQueue->pop();
    $class = $task->getName();
    if (! $container->fetch($class)) {
        $container->regist($class, new $class());
    }
    // fire
    $task->fire();

6.worker平滑重启    

	$proc = new MyMutiProc();
    $proc->restartWorker();
