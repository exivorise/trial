This package is a template/sample code to interact with 5D Robotics Positioning Engine (PE).

**Note - All measurements units use metric system**

## DEPENDENCIES
Operating System - Ubuntu 14.04 LTS
Compiler - C++11 Compiler

## BUILD

Change directory to package directory.

```
$ cd path_to_package/pe_api_sample_code
```

Run *build_package.sh* script

```
$ ./build_package.sh
```
This builds the sample code and installs the executable in the install directory.

## RUN

To run the sample code, run the executable in the install directory with arguments for host computer (PE computer) IP_ADDRESS and PORT.

*Note: By default, port for PE is set to 7000.*

 ```
 $ ./install/pe_interface <IP_ADDRESS> <PORT>
 ```

If IP_ADDRESS and PORT are not passed as arguments to the executable, the IP_ADDRESS is set to 127.0.0.1 and PORT is set to 7000.

__Example Output__
```
$ ./install/pe_interface 127.0.0.1 7000
Server IP: 127.0.0.1
Port: 7000
Comms: Connected
------------------------------------------------------------------------
[WRITE]: Set Odom
Setting Linear Velocity: [0, 0, 0]
Setting Angular Velocity: [0, 0, 0]
Setting Linear Covariance: [1, 0, 0, 0, 1, 0, 0, 0, 1]
Setting Angular Covariance: [1, 0, 0, 0, 1, 0, 0, 0, 1]
------------------------------------------------------------------------
[READ]: Local Pose Info
Position: [56.8917, 17.021, 0]
Localization Status: GOOD
Initialization Status: INITIALIZED
Node Count: 4
...
```

The executable outputs the ip address and port it uses to establish the comms interface and then starts printing out the messages read/written.

## READING MESSAGES

The sample code runs a loop on a thread to read messages from the comms interface and parses them appropriately to read the information contained.

```
// Identify message id contained in header
switch (peHeader->header.msg_id)
{
    case pe_api_msgs::MsgIds::<PE_MESSAGE_ID>:
        // Deserialize data buffer and parse message
        break;
}
```

For example, the __LocalPoseInfo__ message is handled in the sample code,

```
case pe_api_msgs::MsgIds::LOCAL_POSE_INFO:
{
    std::cout << "[PE Read Msgs]: Local Pose Info" << std::endl;
    peMsg = new pe_api_msgs::LocalPoseInfo();

    // Deserialize data buffer from char array to LocalPoseInfo class
    if(!peMsg->deserialize(dataPacket.data, dataPacket.length))
    {
        std::cout << "[PE Read Msgs]: Cannot deserialize pose info" << std::endl;
        break;
    }

    // Parse and output LocalPoseInfo to console
    parseLocalPose(peMsg);
    break;
}
```

The function __parseLocalPose()__ takes the deserialized __PeMsg__ and outputs to console

```
void parseLocalPose(const pe_api_msgs::PeMsg* peMsg)
{
    // Cast generic PeMsg to LocalPoseInfo
    const pe_api_msgs::LocalPoseInfo* poseInfo = static_cast<const pe_api_msgs::LocalPoseInfo*>(peMsg);

    // Output position data to console
    std::cout << "Position: [" << poseInfo->pose.pose.position.x << ", " << poseInfo->pose.pose.position.y << ", " << poseInfo->pose.pose.position.z << "]\n";

    // Output Localization Status to console
    switch (poseInfo->localization_status.simple_status)
    {
        case pe_api_msgs::SimpleLocalizationStatus::BAD:
            std::cout << "Localization Status: BAD\n";
            break;
        case pe_api_msgs::SimpleLocalizationStatus::GOOD:
            std::cout << "Localization Status: GOOD\n";
            break;
        case pe_api_msgs::SimpleLocalizationStatus::UNKNOWN:
            std::cout << "Localization Status: UNKNOWN\n";
            break;
    }

    // Output Initialization status to console
    switch (poseInfo->localization_status.init_status)
    {
        case pe_api_msgs::InitStatus::IDLE:
            std::cout << "Initialization Status: IDLE\n";
            break;
        case pe_api_msgs::InitStatus::INITIALIZING:
            std::cout << "Initialization Status: INITIALIZING\n";
            break;
        case pe_api_msgs::InitStatus::INITIALIZED:
            std::cout << "Initialization Status: INITIALIZED\n";
            break;
    }

    // Output UltraWideBand node count to console
	std::cout << "Node Count: " << int(poseInfo->localization_status.node_count) << std::endl;
}
```

## SETTING ODOMETRY

The sample code sends odometry messages to the PE system by filling in the message fields and sending it over the comms interface,

```
// Set the start flag
setOdomMsg.header.start_flag = pe_api_msgs::MsgIds::START_FLAG;

// Set the message ID - this helps the API to identify this message as an odometry message
setOdomMsg.header.msg_id = pe_api_msgs::MsgIds::SET_ODOM;

// Set the correct message length
setOdomMsg.header.msg_length = setOdomMsg.size();

// Fill in the time stamp for the message
getSysTime(setOdomMsg.header.sys_time_s, setOdomMsg.header.sys_time_ms);

// Fill in linear velocities
setOdomMsg.twist.twist.linear.x = 0.0;
setOdomMsg.twist.twist.linear.y = 0.0;
setOdomMsg.twist.twist.linear.z = 0.0;

// Fill in angular velocities
setOdomMsg.twist.twist.angular.x = 0.0;
setOdomMsg.twist.twist.angular.y = 0.0;
setOdomMsg.twist.twist.angular.z = 0.0;

// Fill in linear and angular covariances
for(unsigned int i = 0; i < 9; i++)
{
    setOdomMsg.twist.linear_covariance[i] = 0.0;
    setOdomMsg.twist.angular_covariance[i] = 0.0;
}
```

## EXAMPLE MESSAGES IN SAMPLE CODE

This package supports the following incoming messages,

* LocalPoseInfo
* SetOdomAck
* GetPeStateAck

This package supports the following outgoing messages,

* SetOdom
* GetPeState
