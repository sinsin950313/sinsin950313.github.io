---
title: "Auto Fire Suppression Turret"
date: 2022-07-11
categories: ["Development", "portfolio"]
tags: ["cpp", "portfolio"]
---
![](/images/fae4244c-0f60-49ab-becb-029598a6abff-image.jpeg)

무인 화재 감지 및 초기 진압을 위한 터렛 입니다.  <br>

프로젝트 당시 소프트웨어 설계와 구현을 맡았습니다.  <br>

프로젝트를 진행하면서 객체지향적 설계, 디자인 패턴 적용을 특히 집중하여 수행하였으며, 물리적으로 분리되어있는 하드웨어에서 각 요소에 필요한 소프트웨어를 설계하는 과정에서 평소 애매하게 이해하고 있던 객체지향성에 대해 조금 더 이해할 수 있는 계기가 되었고, 특히 캡슐화에 대해 직관적으로 이해할 수 있었습니다.  <br><br><br>

# 시스템 구성
시스템는 Observer, Server, Turret 총 3개의 하드웨어로 이루어져 있습니다.  <br>

![](/images/ede921af-8d0d-46d1-98e0-ed456ff06c59-image.png)

프로젝트에서는 1개의 observer, turret으로 이루어져있지만, 확장성을 고려하여 다수의 observer, turret을 이용할 수 있더록 각 하드웨어의 Type을 분류해주는 HardwareType class를 가지고 있습니다.  <br>
각 하드웨어는 크게 Hardware class에서 Generalize되며, 하드웨어간의 통신을 위한 CommunicationModule을 참조하고 있습니다.  <br><br>
  
![](/images/d2258d89-8729-4621-8cbc-26d71f1ba209-image.png)

또한 모든 하드웨어는 Finity State Machine방식으로 구현되어 각각 State를 가지며 하드웨어의 Type마다 다른 State를 갖도록 추상화되어 있습니다.  <br>
  
하드웨어는 action은 Generalize된 Hardware class에 정의되어 있지만, State에 따라 행동이 변경되며, 각 State를 변경하는 클래스로 StateOrder계통의 class가 존재합니다. 그렇기 때문에 main loop를 가지는 HardwareControlSystem은 run()함수만을 수행합니다. 이때 StateOrder는 Singleton방식으로 구현되어 있습니다.  <br><br><br>
  
# 알고리즘 및 통신 절차
![](/images/64fd9b00-9b5c-4a64-8dd1-656489a7377e-image.png)

Detector와 Turret이 분리되어 있기 때문에 모든 하드웨어는 Server와의 통신을 통해 정확한 화재 지점에 대한 정보를 전파받아야 합니다.  <br><br>
![](/images/73cb93a5-47fe-4481-b7e4-ba3f4f9a7228-image.png)
![](/images/b84962ff-ae3b-4242-ac64-53d1ed5386f6-image.png)

통신을 위한 패킷은 확장성을 고려하여 설계하였습니다.<br><br>


```cpp
int CommunicationModule::setPacketHeader(bool ACK, bool separate, char seed, char dstAddress)
{
    _packetBuffer[0] = START_CLOCK;
    _packetBuffer[1] = _order->getSignalFlag();
    _packetBuffer[2] = getMyAddress();
    _packetBuffer[3] = dstAddress;
    _packetBuffer[4] = _order->getOrderFlag();

    char tmp = 0b00000000;
    if(ACK)
	tmp = tmp | 0b00000001;
    if(separate)
	tmp = tmp | 0b00000010;
    else
	tmp = tmp | 0b00000100;

    _packetBuffer[5] = tmp;
    _packetBuffer[6] = seed;

    return 7;
}

void CommunicationModule::setErrorDetectingCode()
{
    char parity = _packetBuffer[0];

    for(int i = 1; i < (BUFFER_SIZE - 1); i++)
	parity = _packetBuffer[i] ^ parity;

    _packetBuffer[BUFFER_SIZE - 1] = parity;
}
```

통신을 위해 패킷에 필요한 데이터는 직렬화하고, 마지막 행에 parity비트를 응용한 byte를 추가하여 데이터의 정합성을 보장하는 과정을 거쳤습니다.<br><br>

```cpp
void BluetoothCommunicationModule::read(char* const* data)
{
    CustomIO::output("Receiving...");
    char c = 0;
    do
    {
	c = _receiver->read();

#if defined(DEBUG)
	CustomIO::output(c, true);
	/*
	CustomIO::output(START_CLOCK ^ c, true);
	bool b = START_CLOCK ^ c != 0;
	CustomIO::output(b);
	b = START_CLOCK ^ c != 0b00000000;
	CustomIO::output(b);
	*/
#endif

    } while(START_CLOCK != c );

    (*data)[0] = c;

    int i = 1;
    while(i < BUFFER_SIZE)
    {
	c = _receiver->read();
	(*data)[i] = c;
	i++;
    }

    delay(500);

    while(_receiver->isConnected())
    {
#if defined(DEBUG)
	CustomIO::output("Bluetooth - send Response");
	CustomIO::output(_privateAnswerSignal);
#endif
	_receiver->write(_privateAnswerSignal);
	delay(500);
    }

#if defined(DEBUG)
    CustomIO::output("===================================");
    CustomIO::output("Reception Finish");
    i = 0;
    while(i < BUFFER_SIZE)
    {
	CustomIO::output((*data)[i], true);
	i++;
    }
    CustomIO::output("===================================");
#endif
}
```

또한 Three-way-handshake를 수행하여 수신자가 확실하게 데이터를 받았는지를 보장했습니다.<br><br><br>
  
# Observer
![](/images/04400d6c-6d7b-4070-b29d-812cc7c9d467-image.png)

자동으로 화재를 감지, 좌표를 측정하는 Observer입니다.<br><br>
![](/images/74866003-51c2-4cbd-950e-5b05987f5343-image.png)

![](/images/9ec22783-d2ce-4d4e-a898-bfd022c06e0e-image.png)

Observer는 크게 상위 행동을 관리하는 Observer class와 말단 하드웨어를 직접 작동하는 ObserverDriveModule class를 갖고 있습니다. 이 과정을 통해 시스템 요소인 Observer와 이를 이루는 하위 하드웨어간의 개념적 분리와 실제 하드웨어 작동에 대한 추상화를 구현하였습니다.<br><br><br>
  
# Server
Detector에서 감지된 열원의 좌표는 Server로 전송되어 연산과정을 거친 후 Turret에게 적절한 포구초속과 방향을 전달해 줍니다.
![](/images/4f5b0589-e8b8-442a-ae41-3a52fce090e0-image.png)
![](/images/87e91353-2ec5-4552-8465-f02aff166f3a-image.png)
![](/images/ef7455a3-60c9-4a64-9578-59ac078a9b7b-image.png)
![](/images/67a29f04-1b91-4007-b5b6-82fd3320ff1a-image.png)
![](/images/2902e4a6-361e-471b-ba45-b0794ba2282a-image.png)

```cpp
int Server::fire()
{
#if defined(DEBUG)
    output("Fire State");
#endif
    char observerID = _observerCM->getSourceID();
    if(_observerCM->getTransmittedData().getDataType() == CommonData::DataType::SHPERICAL)
    {
	ShpericalData* pShp = reinterpret_cast<ShpericalData*>(&_observerCM->getTransmittedData());
	//this code maybe changed if observer install condition is changed
	//now observer installed right way
	ShpericalData fireShp(pShp->getYawAngle(), pShp->getPitchAngle(), pShp->getRadius());
	//fireShp.setPitchAngle(-fireShp.getPitchAngle());

#if defined(DEBUG)
	output("Observer To Server : Fire Shperical Coordinate");
	fireShp.print();
#endif

	CoordinateData relativeFireCoord;
	CoordinateData absFireCoord;
	ShpToCoord(&fireShp, &relativeFireCoord);
#if defined(DEBUG)
	output("Relative Fire Coordinate From Observer");
	relativeFireCoord.print();
#endif
	RelativeCoordToAbsCoord(_hpData.getGlobalPosition(observerID), &relativeFireCoord, &absFireCoord);

#if defined(DEBUG)
	output("Absolute Observer Coordinate");
	_hpData.getGlobalPosition(observerID)->print();
	output("Absolute Fire Coordinate");
	absFireCoord.print();
#endif

	//how can i control only one observer with multiply turret?
	char turretID = findOptimumTurretID();
	ShpericalData obsShp;
	bool confirm = requestTurretCheck(turretID, &absFireCoord, &obsShp);

	if(!confirm)
	    return ServerStateList::FIRE;

#if defined(DEBUG)
	output("Turret To Server : Obstacle Shperical Coordinate");
	obsShp.print();
#endif

	CoordinateData relObsCoord;
	ShpToCoord(&obsShp, &relObsCoord);
	CoordinateData absObsCoord;
	RelativeCoordToAbsCoord(_hpData.getGlobalPosition(turretID), &relObsCoord, &absObsCoord);
#if defined(DEBUG)
	output("Server, Fire State : need rewrite this code. Obstacle does not considered yet");
#endif
	int obsHeight = absObsCoord.getZ();
	absObsCoord.setZ(0);

#if defined(DEBUG)
	output("Absolute Obstacle Coordinate");
	absObsCoord.print();
	output("Obstacle Height");
	output(obsHeight);
#endif

	ShpericalData aimShp;
	optimumPathAlgorithm(turretID, &absFireCoord, &absObsCoord, obsHeight, &aimShp);

#if defined(DEBUG)
	output("Optimum Data : Velocity, Yaw, Pitch");
	aimShp.print();
#endif
	while(_observerCM->getOrder() != CommunicationSignalList::EXTINGUISH)
	{
	    requestTurretLaunch(turretID, &aimShp);

#if defined(DEBUG)
	output("Wait For Turret Launch Signal");
#endif
	    _turretCM->receive();

	    requestObserverCheck(observerID);

#if defined(DEBUG)
	output("Wait For Observer Impact Area Check Response");
#endif
	    _observerCM->receive();
	}

	return ServerStateList::EXTINGUISH;
    }

    return ServerStateList::FIRE;
}
```

당시 디버깅을 위해 log를 출력하거나, 하드웨어의 설정을 변경하는 상황이 잦았는데, 이를 쉽게 하기 위해 전처리 구문을 이용한 코드 변경 기법을 적용하여 높은 생산성을 이끌어 낼 수 있었습니다.<br><br>

```cpp
//Server.cpp
	ShpericalData* pShp = reinterpret_cast<ShpericalData*>(&_observerCM->getTransmittedData());
//CommunicationModule.h
	Data & getTransmittedData() { return *_dataPointer; }
```

또한 데이터 전송 과정에서 객체의 생존 주기를 확실히 하기 위하여 RAII원칙에 입각한 데이터 관리를 수행하였습니다.<br><br><br>

# Turret
![](/images/c3f603da-dfcd-4f80-835d-df20f2a6146a-image.jpeg)


![](/images/fbfc2355-c5cd-4cab-8258-487697a9341c-image.png)

![](/images/183fedbd-34f5-4981-bb89-5209f5e49deb-image.png)

Turret 역시 개념론적인 동작을 관리하는 Turret class와 실질적인 동작을 관리하는 TurretDriveModule class로 분리하였습니다.<br><br><br>

# 작동영상
{% include embed/youtube.html id='C2GlhAOUUfk' %}



[Source](https://github.com/sinsin950313/Auto-Fire-Suppression-Turret)
