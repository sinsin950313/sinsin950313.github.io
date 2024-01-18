---
title: "BlockChain Used as DataBase"
date: 2022-07-11
categories: ["Development", "portfolio"]
tags: ["blockchain", "portfolio", "smart contract"]
---
![](/images/bf3a4a38-ac7b-4b6f-a4fa-5bc06b9c36c3-image.png)

본 프로젝트에서는 Smart Contract를 응용하여 Serverless Architecture DataBase를 구현해보았습니다.<br><br>

# 개요
고전적인 Server-Client구조에서 발생하는 게임 내 재화 문제의 원인 중 하나는 Server와 Client사이의 동기화 시점에 의한 문제였습니다.

![](/images/ff4429ab-eb8a-48bb-8d69-c358bf551f9e-image.png)

플레이어의 모든 순간을 매번 Server에 동기화할 순 없기 때문에 이를 해결하기 위해선 개발자의 섬세한 동기화 시점 선택을 요구했습니다.<br><br><br>

![](/images/b8713fa9-3f26-4da6-a06e-6c1d33d07c9f-image.png)

이러한 부분에 있어서, '공개 장부'라는 블록체인의 특성과, '프로그래밍 가능'이라는 Smart Contract의 특성을 이용한다면 동기화 시점에 대한 고려가 필요없는 DataBase Server의 역할을 구현할 수 있지 않을까라는 궁금증을 가지고 개발을 시도해봤습니다.<br><br>

프로젝트 진행 과정에서 전반적인 설계와 Server-Client간의 통신을 주로 구현했는데, Rest 방식을 이용하여 통신을 구현한 것이 인상적이었습니다.<br><br>

또한, 처음으로 프로그래밍 가능한 동료들과의 협업이었기 때문에, 세심하게 상하위 설계를 마치고 실개발에 들어간 것이 흥미로운 경험이었습니다.<br><br><br>

# UseCase
![](/images/0e284908-5bed-44bf-b5c8-9fbbb2e064be-image.png)
![](/images/766cb1ba-6e33-4519-86ea-46d080bf18c2-image.png)
![](/images/63d19925-c8ca-4e73-87ec-94ea31677102-image.png)

게임 자체의 완성도가 중요한 것은 아니었으므로, 간단하게 아이템 구매, 선물, 게임 기록 업데이트 3가지 상황만 가정하여 구현했습니다.<br><br>

# 설계
![](/images/dac8d467-bb6b-406f-93fa-8714bfcdc46d-image.png)

![](/images/d5f5a1dd-7c7a-4fbd-866c-e18e03d1768b-image.png)

상위 설계의 경우, 다른 프로젝트나, 개인적인 개발 중에도 여러번 진행해봤지만, 각 객체의 기능과 Interface를 미리 설계하는 것은 생각보다 어려운 작업이었습니다.<br><br><br>

# 코드
```cpp
pragma solidity ^0.4.18;

contract runner{

    struct Player {
	uint user_index;
	uint high_score; // high score remove
	uint[3] item_list;
	uint reward;
	bool success;
    }
    Player[] public players;

    // item list 
    uint[3] public init_item_list = [0,0,0];
    uint[3] public item_price = [100,200,300]; // add item price to calculate buy function

    function getPlayer_high_score(uint _index) public view returns (uint){
	return players[_index].high_score;
    }
    function getPlayer_item_list(uint _index) public view returns (uint[3]){
	return players[_index].item_list;
    }
    function getPlayer_reward(uint _index) public view returns (uint){ 
	return players[_index].reward;
    }

```
Smart Contract 내부에 Player 정보 객체를 저장한 후, 조회 및 갱신 때마다 Smart Contract 객체에 접근하는 방식으로 구현했습니다.<br><br>

```cs
    public bool buy(int userIndex, int[] itemCount)
    {
	callRest = post;
	bool success = true;

	int i = 0;
	do
	{
	    BuyClass obj = new BuyClass(userIndex, i, itemCount[i]);
	    string str = JsonUtility.ToJson(obj);
	    body.inputs = str;
	    str = JsonUtility.ToJson(body);
	    parameter = multiJsonProcessing(str);

	    func = "buy";
	    transaction();

	    success = isRecentTradeSuccess(userIndex);
	    i++;
	} while(i < itemCount.Length && success);

	return success;
    }
    private void transaction()
    {
	callURI = transactionURI + func;
	LogPrinter.getInstance().print("URI : " + callURI);
	LogPrinter.getInstance().print("Rest API : " + callRest);
	LogPrinter.getInstance().print("Parameters : " + parameter);

	UploadHandler up = new UploadHandlerRaw(Encoding.UTF8.GetBytes(parameter));
	up.contentType = Content_Type;

	DownloadHandler down = new DownloadHandlerBuffer();

	UnityWebRequest transactionRequest = new UnityWebRequest(callURI, callRest, down, up);
	transactionRequest.SetRequestHeader("Authorization", Authorization);
	LogPrinter.getInstance().print("Encoding Data : " + Encoding.Default.GetString(up.data));

	UnityWebRequestAsyncOperation res = transactionRequest.SendWebRequest();
	while(!res.isDone);

	if (transactionRequest.isNetworkError || transactionRequest.isHttpError)
	    LogPrinter.getInstance().print("Transaction Request Error : " + transactionRequest.error + down.text);
	else
	{
	    responseStr = down.text;
	    LogPrinter.getInstance().print("Transaction Response : " + responseStr);
	}
    }

```

통신을 담당하는 CommunicationModule은 Singleton 패턴으로 구현되었으며, 모든 통신은 Rest API를 이용하여 수행되었습니다.<br><br>

```cs
	int count = 1;
	while(arr[end] != ']')
	{
	    if(arr[end] == ',')
		count++;
	    end++;
	}
	_itemList = new int[count];
	int arrayPointer = 0;
	while(start < end)
	{
	    if(arr[start] == '"')
	    {
		start++;
		int endPointer = start + 1;
		while(arr[endPointer] != '"')
		    endPointer++;
		int length = endPointer - start;
		temp = new char[length];
		int i_ = 0;
		while(i_ < length)
		{
		    temp[i_] = arr[start + i_];
		    i_++;
		}
		tempStr = new string(temp);
		_itemList[arrayPointer] = int.Parse(tempStr);
		arrayPointer++;

		start = endPointer + 1;
	    }
	    start++;
	}

```
CommunicationModule을 통해 받아온 Serialize된 데이터는 Properties에서 DeSerialize 과정을 거쳐 Client에서 Game Play에 필요한 정보를 갱신합니다.<br><br><br>

# 결과
#### Play

{% include embed/youtube.html id='ThUGTnFtTj0' %}

#### Buy

{% include embed/youtube.html id='1ou_eQwEk0E' %}

#### Send

{% include embed/youtube.html id='zXzwJ044bKI' %}

#### Transaction

![](/images/eba300e5-7243-40b5-b77a-f4c1051ec85b-image.png)

![](/images/109fbc53-b616-46c8-826c-0f6442c7ce60-image.png)

![](/images/665b9b4f-63e5-4874-9639-a708b5a829b3-image.png)

![](/images/a3beb61d-cadc-45ba-ad5d-c7e915561e68-image.png)

게임 플레이 전후로 Transaction이 발생한 것을 확인할 수 있습니다.<br><br>

마지막으로 활동 영상입니다.

{% include embed/youtube.html id='_iiJBtuhorQ' %}

[Source](https://github.com/sinsin950313/BlockChain-used-as-DataBase)
