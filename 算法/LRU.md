## :LRU(cache)

“ 手把手嘴对嘴用教你写一个LRU缓存淘汰算法。---鲁迅”


###LRU介绍


**缓存是一种提高数据读取性能的技术。但是对于计算机来说，并不可能缓存所有的数据，在达到它的临界空间时,我们需要通过一些规则用新的数据取代掉一部分的缓存数据。这时候你会如果选择替换呢？**

**替换的策略有很多种，常用的有以下几种:**

FIFO(先进先出策略)

LFU(最少使用策略)

LRU(最近最少使用策略)

NMRU(在最近没有使用的缓存中随机选择一个替换)



**介于我这篇主要实现LRU，所以就不去介绍其他的了，可以自行去了解。**


**假设你已经有5个女朋友了，此时你成功勾搭上一个新女朋友，在你沉迷女色的同时，你惊奇的发现，你已经不能像年轻时一样以一敌六了，你必须舍弃若干个女朋友，这时候，身拥六个女朋友的渣男你，彻底展示出你的渣男本色，和最近最少秀恩爱的小姐姐说再见：“对不起，国篮此时需要我挺身发边线球，我楠辞琦咎，再见。”，就这样在你成功勾搭一个新小姐姐，你的身体临界点的同时,你就必须舍弃其他的小姐姐。**

**下面来张实际点的图搞清楚他的原理。**

<a href="https://github.com/wuqinqiang/">
​    <img src="https://github.com/wuqinqiang/Lettcode-php/blob/master/images/LRU.png">
</a> 


**基于上述图片，我们知道，对于LRU的操作，无非在于插入(insert),删除(delete)，以及替换，针对替换来说，如果缓存空间满了，那么就是insert to head and delete for tail。如果未满，也分为两种，一种是缓存命中的话，只需要把缓存的值move to head。如果之前不存在，那么就是 insert to head。**




###实现过程



**接下来就是数据结构的选择了。数组的存储是连续的内存空间，虽然查询的时间复杂度是O(1),但是删除和插入为了保存内存空间的连续性，需要进行搬移，那么时间复杂度就是O(n),为了实现能快速删除，故而采用双向链表。但是链表的查询时间复杂度是O(n),那么就需要hash table。屁话说了这么多，代码实现。其实之前刷过这道题目。特地拿出来讲一下。**

```php

class LRUCache {
    private $capacity;
    private $list;
    /**
     * @param Integer $capacity
     */
    function __construct($capacity) {
        $this->capacity=$capacity;
        $this->list=new HashList();
        
    }
  
    /**
     * @param Integer $key
     * @return Integer
     */
    function get($key) {
        if($key<0) return -1;
        return $this->list->get($key);
    }
  
    /**
     * @param Integer $key
     * @param Integer $value
     * @return NULL
     */
    function put($key, $value) {
        $size=$this->list->size;
        $isHas=$this->list->checkIndex($key);
        if($isHas || $size+1 > $this->capacity){
            $this->list->removeNode($key);
        }
        $this->list->addAsHead($key,$value);
    }
}



class HashList{
    public $head;
    public $tail;
    public $size;
    public $buckets=[];
    public function __construct(Node $head=null,Node $tail=null){
        $this->head=$head;
        $this->tail=$tail;
        $this->size=0;
    }
    
    //检查键是否存在
    public function checkIndex($key){
        $res=$this->buckets[$key];
        if($res){
            return true;
        }
        return false;
    }
    
    public function get($key){
        $res=$this->buckets[$key];
        if(!$res) return -1;
        $this->moveToHead($res);
        return $res->val;
    }
    
    //新加入的节点
    public function addAsHead($key,$val)
{
        $node=new Node($val);
        if($this->tail==null && $this->head !=null){
            $this->tail=$this->head;
            $this->tail->next=null;
            $this->tail->pre=$node;
        }
        $node->pre=null;
        $node->next=$this->head;
        $this->head->pre=$node;
        $this->head=$node;
        $node->key=$key;
        $this->buckets[$key]=$node;
        $this->size++;
    }
    
    //移除指针(已存在的键值对或者删除最近最少使用原则)
    public function removeNode($key)
{
        $current=$this->head;
        for($i=1;$i<$this->size;$i++){
            if($current->key==$key) break;
            $current=$current->next;
        }
        unset($this->buckets[$current->key]);
        //调整指针
        if($current->pre==null){
            $current->next->pre=null;
            $this->head=$current->next;
        }else if($current->next ==null){
            $current->pre->next=null;
            $current=$current->pre;
            $this->tail=$current;
        }else{
            $current->pre->next=$current->next;
            $current->next->pre=$current->pre;
            $current=null;
        }
        $this->size--;
        
    }
    
    //把对应的节点应到链表头部(最近get或者刚刚put进去的node节点)
    public function moveToHead(Node $node)
{
        if($node==$this->head) return ;
        //调整前后指针指向
        $node->pre->next=$node->next;
        $node->next->pre=$node->pre;
        $node->next=$this->head;
        $this->head->pre=$node;
        $this->head=$node;
        $node->pre=null;
    }
    
    
}


class Node{
    public $key;
    public $val;
    public $next;
    public $pre;
    public function __construct($val)
{
        $this->val=$val;
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * $obj = LRUCache($capacity);
 * $ret_1 = $obj->get($key);
 * $obj->put($key, $value);
 */
 
 ```
****

<a href="https://github.com/wuqinqiang/">
​    <img src="https://github.com/wuqinqiang/Lettcode-php/blob/master/qrcode_for_gh_c194f9d4cdb1_430.jpg" width="150px" height="150px">
</a> 
   
    
    
    

