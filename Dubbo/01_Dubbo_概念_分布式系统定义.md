---
title: 01_Dubbo_概念_分布式系统定义
date: 2019-09-08 12:23:17
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 01_Dubbo\_概念_分布式系统定义

## 基础知识

### 分布式基础理论

#### 什么是分布式系统

《分布式系统原理与范型》定义：

“分布式系统是若干独立计算机的集合，这些计算机对于用户来说就像单个相关系统”

分布式系统（distributed system）是建立在网络之上的软件系统。



随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需**一个治理系统**确保架构有条不紊的演进。





<ul class="list-box"><li class="watched on"><a href="/video/av30612478/?p=1" class="" title="01、尚硅谷_Dubbo_概念_分布式系统定义"><i class="van-icon-videodetails_play"></i><span class="s1">P1</span>01、尚硅谷_Dubbo_概念_分布式系统定义
        </a></li><li class=""><a href="/video/av30612478/?p=2" class="" title="02、尚硅谷_Dubbo_概念_应用的架构演变"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P2</span>02、尚硅谷_Dubbo_概念_应用的架构演变
        </a></li><li class=""><a href="/video/av30612478/?p=3" class="" title="03、尚硅谷_Dubbo_概念_RPC简介"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P3</span>03、尚硅谷_Dubbo_概念_RPC简介
        </a></li><li class=""><a href="/video/av30612478/?p=4" class="" title="04、尚硅谷_Dubbo_概念_简介"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P4</span>04、尚硅谷_Dubbo_概念_简介
        </a></li><li class=""><a href="/video/av30612478/?p=5" class="" title="05、尚硅谷_Dubbo_概念_设计架构"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P5</span>05、尚硅谷_Dubbo_概念_设计架构
        </a></li><li class=""><a href="/video/av30612478/?p=6" class="" title="06、尚硅谷_Dubbo_环境搭建_ZooKeeper注册中心"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P6</span>06、尚硅谷_Dubbo_环境搭建_ZooKeeper注册中心
        </a></li><li class=""><a href="/video/av30612478/?p=7" class="" title="07、尚硅谷_Dubbo_环境搭建_管理控制台"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P7</span>07、尚硅谷_Dubbo_环境搭建_管理控制台
        </a></li><li class=""><a href="/video/av30612478/?p=8" class="" title="08、尚硅谷_Dubbo_环境搭建_创建提供者消费者工程"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P8</span>08、尚硅谷_Dubbo_环境搭建_创建提供者消费者工程
        </a></li><li class=""><a href="/video/av30612478/?p=9" class="" title="09、尚硅谷_Dubbo_服务提供者配置&amp;测试"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P9</span>09、尚硅谷_Dubbo_服务提供者配置&amp;测试
        </a></li><li class=""><a href="/video/av30612478/?p=10" class="" title="10、尚硅谷_Dubbo_服务消费者配置&amp;测试"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P10</span>10、尚硅谷_Dubbo_服务消费者配置&amp;测试
        </a></li><li class=""><a href="/video/av30612478/?p=11" class="" title="11、尚硅谷_Dubbo_监控中心_Simple Monitor安装配置"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P11</span>11、尚硅谷_Dubbo_监控中心_Simple Monitor安装配置
        </a></li><li class=""><a href="/video/av30612478/?p=12" class="" title="12、尚硅谷_Dubbo_与SpringBoot整合"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P12</span>12、尚硅谷_Dubbo_与SpringBoot整合
        </a></li><li class=""><a href="/video/av30612478/?p=13" class="" title="13、尚硅谷_Dubbo_配置_dubbo.properties&amp;属性加载顺序"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P13</span>13、尚硅谷_Dubbo_配置_dubbo.properties&amp;属性加载顺序
        </a></li><li class=""><a href="/video/av30612478/?p=14" class="" title="14、尚硅谷_Dubbo_配置_启动检查"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P14</span>14、尚硅谷_Dubbo_配置_启动检查
        </a></li><li class=""><a href="/video/av30612478/?p=15" class="" title="15、尚硅谷_Dubbo_配置_超时&amp;配置覆盖关系"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P15</span>15、尚硅谷_Dubbo_配置_超时&amp;配置覆盖关系
        </a></li><li class=""><a href="/video/av30612478/?p=16" class="" title="16、尚硅谷_Dubbo_配置_重试次数"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P16</span>16、尚硅谷_Dubbo_配置_重试次数
        </a></li><li class=""><a href="/video/av30612478/?p=17" class="" title="17、尚硅谷_Dubbo_配置_多版本"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P17</span>17、尚硅谷_Dubbo_配置_多版本
        </a></li><li class=""><a href="/video/av30612478/?p=18" class="" title="18、尚硅谷_Dubbo_配置_本地存根"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P18</span>18、尚硅谷_Dubbo_配置_本地存根
        </a></li><li class=""><a href="/video/av30612478/?p=19" class="" title="19、尚硅谷_Dubbo_配置_与SpringBoot整合的三种方式"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P19</span>19、尚硅谷_Dubbo_配置_与SpringBoot整合的三种方式
        </a></li><li class=""><a href="/video/av30612478/?p=20" class="" title="20、尚硅谷_Dubbo_高可用_ZooKeeper宕机与Dubbo直连"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P20</span>20、尚硅谷_Dubbo_高可用_ZooKeeper宕机与Dubbo直连
        </a></li><li class=""><a href="/video/av30612478/?p=21" class="" title="21、尚硅谷_Dubbo_高可用_负载均衡机制"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P21</span>21、尚硅谷_Dubbo_高可用_负载均衡机制
        </a></li><li class=""><a href="/video/av30612478/?p=22" class="" title="22、尚硅谷_Dubbo_高可用_服务降级"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P22</span>22、尚硅谷_Dubbo_高可用_服务降级
        </a></li><li class=""><a href="/video/av30612478/?p=23" class="" title="23、尚硅谷_Dubbo_高可用_服务容错&amp;Hystrix"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P23</span>23、尚硅谷_Dubbo_高可用_服务容错&amp;Hystrix
        </a></li><li class=""><a href="/video/av30612478/?p=24" class="" title="24、尚硅谷_Dubbo_原理_RPC&amp;Netty原理"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P24</span>24、尚硅谷_Dubbo_原理_RPC&amp;Netty原理
        </a></li><li class=""><a href="/video/av30612478/?p=25" class="" title="25、尚硅谷_Dubbo_原理_框架设计"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P25</span>25、尚硅谷_Dubbo_原理_框架设计
        </a></li><li class=""><a href="/video/av30612478/?p=26" class="" title="26、尚硅谷_Dubbo_原理_标签解析"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P26</span>26、尚硅谷_Dubbo_原理_标签解析
        </a></li><li class=""><a href="/video/av30612478/?p=27" class="" title="27、尚硅谷_Dubbo_原理_服务暴露流程"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P27</span>27、尚硅谷_Dubbo_原理_服务暴露流程
        </a></li><li class=""><a href="/video/av30612478/?p=28" class="" title="28、尚硅谷_Dubbo_原理_服务引用流程"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P28</span>28、尚硅谷_Dubbo_原理_服务引用流程
        </a></li><li class=""><a href="/video/av30612478/?p=29" class="" title="29、尚硅谷_Dubbo_原理_服务调用流程"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P29</span>29、尚硅谷_Dubbo_原理_服务调用流程
        </a></li><li class=""><a href="/video/av30612478/?p=30" class="" title="30、尚硅谷_Dubbo_结束语"><i class="van-icon-videodetails_play" style="display: none;"></i><span class="s1">P30</span>30、尚硅谷_Dubbo_结束语
        </a></li></ul>