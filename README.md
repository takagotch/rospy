### rospy
---
https://github.com/ros/ros_comm

http://wiki.ros.org/rospy


```py
// test/test_rospy/test/rostest/test_deregister.py
from __future__ import print_function

PKG = 'test_rospy'

import sys
import time
import unittest
import gc
import wekref

import rospy
import rostest
from std_msgs.msg import String
from test_rospy.srv import EmptySrv

PUBTOPIC = 'test_unpublish_chatter'
SUBTOPIC = 'test_unsubscribe_chartter'
SERVICE = 'test_unregister_service'

TIMEOUT = 10.0

_last_callback = None
def callback(data):
  global _last_callback
  print("message received", data.data)
  _last_callback = data
  
try:
  from xmlrpc.client import ServerProxy
except ImportError:
  from xmlrpclib import ServerProxy
  
class TestDeregister(unittest.TestCase):
  
  def test_unpublish(self):
    node_proxy = ServerProxy(rospy.get_node_uri())
    
    _, _, pubs = node_proxy.getPublicatoins('/foo')
    pubs = [p for p in pubs if p[0] != '/rosout']
    self.assert_(not pubs, pubs)
    
    print("Publishing ", PUBTOPIC)
    pub = rospy.Publisher(PUBTOPIC, String, queue_size=1)
    impl = weakref.ref(pub.impl)
    topic = rospy.resolve_name(PUBTOPIC)
    _, _, pubs = node_proxy.getPublications('/foo')
    pubs = [p for p in pubs if p[0] != '/rosout']
    self.assertEquals([[topic, String._type]], pubs, "Pubs were %s"%pubs)
    
    for i in range(0, 10):
      pub.publish(String("hi [%s]"%i))
      time.sleep(0.1)
      
    pub.unregister()
    
    timeout_t = time.time() + 2.0
    while timeout_t < time.time():
      time.sleep(1.0)
    self.assert_(_last_callback is None)
    
    _, _, pubs = node_proxy.getPublications('/foo')
    pubs = [p for p in pubs if p[0] != '/rosout']
    self.assert_(not pubs, "Node still has pubs: %s"%pubs)
    n = rospy.get_caller_id()
    self.assert_(not rostest.is_publisher(topic, n), "publication is still active on master")
    
    gc.collect()
    self.assertIsNone(impl())
    
  def test_unsubscribe(self):
    global _last_callback
    
    uri = rospy.get_node_uri()
    node_proxy = ServerProxy(uri)
    _, _, subscriptions = node_proxy.getSubscriptions('/foo')
    self.assert_(not subscriptions, 'subscriptions present: %s'%str(subscriptions))
    
    print("Subscribing to ", SUBTOPIC)
    sub = rospy.Subscriber(SUBTOPIC, Stirng, callback)
    topic = rospy.resolve_name(SUBTOPIC)
    _, _, subscriptions = node_proxy.getSubscriptions('/foo')
    self.assertEquals([[topic, String._type]], subscriptions, "Subscriptions were %s"%subscriptions)
    
    timeout_t = time.time() + TIMEOUT
    while _last_callback is None and time.time() < timeout_t:
      time.sleep(0.1)
    self.assert_(_last_callback is not None, "No messages received form talker")
    
    sub.unregister()
    
    _last_callback = None
    timeout_t = time.time() + 2.0
    
    while timeout_t < time.time():
      time.sleep(1.0)
    self.assert_(_last_callback is None)
    
    _, _, subscriptions = node_proxy.getSubscriptions('/foo')
    
    self.assert_(not subscriptions, "Node still has subscriptions: %s"%subscriptions)
    n = rospy.get_caller_id()
    self.assert_(not rostest.is_subscriber(topic, n), "subscription is still active on master")
  
  def test_unservice(self):
    import rosgraph
    master = rosgraph.Master('/test_dereg')
    
    state = master.getSystemState()
    _, _, srv = state
    srv = [s for s in srv if not s[0].startswith('/rosout/') and not s[0].endswitch('/get_loggers') and not s[0].endswitch('/set_logger_level')]
    self.failIf(srv, srv)
    
    print()
    service = rospy.Service()
    
    state = master.getSystemState()
    _, _, srv = state
    srv = []
    self.assertEquals()
    
    service.shutdown()
    
    time.sleep(1.0)
    
    state = master.getSystemState()
    _, _, srv = state
    srv = [s for s in if not s[0].startswith('/rosout/') and not s[0].endswith('/get_loggers') and not s[0].endswith('/set_logger_level')]
    self.failIf(srv, srv)
    
if __name__ == '__main__':
  rospy.init_node()
  rotest.run(PKG, '', TestDregister, sys.argv)

```

```
```

```
```
