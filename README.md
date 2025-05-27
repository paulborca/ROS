# ROS
import pyads
import rospy
from geometry_msgs.msg import Twist

AMS_NET_ID = '5.19.158.1.1.1'  # Match this to what TwinCAT uses
IP_ADDRESS = '192.168.0.10'     # Host IP

# Command mapping
CMD_STOP = 0
CMD_FORWARD = 1
CMD_BACKWARD = 2
CMD_LEFT = 3
CMD_RIGHT = 4

vel_linear = 0.5
vel_angular = 1.0

def ads_listener():
    plc = pyads.Connection(AMS_NET_ID, pyads.PORT_TC3PLC1, IP_ADDRESS)
    plc.open()

    rospy.init_node('twincat_controller')
    pub = rospy.Publisher('/cmd_vel', Twist, queue_size=10)
    rate = rospy.Rate(10)

    while not rospy.is_shutdown():
        try:
            cmd = plc.read_by_name('MAIN.move_cmd', pyads.PLCTYPE_INT)
            twist = Twist()

            if cmd == CMD_FORWARD:
                twist.linear.x = vel_linear
            elif cmd == CMD_BACKWARD:
                twist.linear.x = -vel_linear
            elif cmd == CMD_LEFT:
                twist.angular.z = vel_angular
            elif cmd == CMD_RIGHT:
                twist.angular.z = -vel_angular
            # else = stop

            pub.publish(twist)
        except Exception as e:
            rospy.logerr(f"ADS Read Error: {e}")
        rate.sleep()

    plc.close()

if __name__ == '__main__':
    try:
        ads_listener()
    except rospy.ROSInterruptException:
        pass
