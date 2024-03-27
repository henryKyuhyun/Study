# Node.js 에서 Winston

### Winston 이란?

Winston은 Node.js에서 가장 널리 사용되는 로깅 라이브러리 중 하나입니다. Node.js의 단순 console.error 와 비교할 때, Winston 은 밑에서 살펴본 것과 같이 몇가지 특징적인 장점이 있습니다.

### Login Library ⇒

![Screenshot 2024-03-07 at 11 57 29 am](https://github.com/henryKyuhyun/Study/assets/118201123/d7d1e741-30d9-4b30-985c-b3d66b4479ef)


### Winston 의 장점 대비 Console.error

- 로그 레벨 구분 : 다양한 로그 레벨을 설정(ex) info, warn, error 등) 하여 로그를 좀 더 세분화하여 관리할 수 있습니다. (참조 : [https://www.coreycleary.me/should-you-use-a-logging-framework-or-console-log-in-node/](https://www.coreycleary.me/should-you-use-a-logging-framework-or-console-log-in-node/))
- 로그의 형식화  :로그 메시지의 형식을 사용자가 정의하여 로깅할 수 있습니다.
- 다양한 출력 옵션(Transports) : 로그를 콘솔 뿐 아니라 파일, 데이터베이스 등 다양한 곳으로 내보낼 수 있습니다.  (참고 : [https://reflectoring.io/node-logging-winston/](https://reflectoring.io/node-logging-winston/))
- 우수한 성능 : 장기적인 로그 관리 및 대규모 어플리케이션의 성능 면에서도 월등 (참고 : [https://betterstack.com/community/guides/logging/nodejs-logging-best-practices/](https://betterstack.com/community/guides/logging/nodejs-logging-best-practices/))

## What console does?

⇒ While browsers implement **`console`** differently, in Node the **`console`** module will print to **`stdout`(Standard output)** and/or **`stderr` (Standard error)**. If you're using **`console.log()`** it will print to **`stdout`** and if you're using **`console.error()`** it will print to **`stderr`**.

Why does this matter ? 

Being able to write to **`stdout`** and **`stderr`** means that you can have Docker or logstash or whichever tool you're using easily pick up these logs. **`stdout`** and **`stderr`** on linux are pipeable, so this becomes easy.

By only using **`console`**, you're reducing a dependency (and all of it's dependencies) that you would have if you were using a logging framework. You don't even have to require/import the module like you do with some other native Node modules like **`fs`**.

*Side note: above **`console`** refers to the **global** console, but it is possible to import console as a Class, which you can then instantiate to configure your own output streams rather than just **`stdout`** and **`stderr`**. I'm pointing this out just as a form of technical due diligence but it is not something you need to be concerned with for now as this is not the way console is usually used in Node. If you'd like to read more about the instantiation method however, you can [check out the docs here](https://nodejs.org/api/console.html#console_class_console).*

Lastly, because it's common for front-end JavaScript developers to work on the Node parts of applications too, **`console`** has the same API methods as the console used by browsers, making it effortless to pick up.

```jsx
console.log() --> writes to stdout
console.debug() --> writes to stdout
console.info() --> writes to stdout

console.error() --> writes to stderr
console.warn() --> writes to stderr
```

```jsx
// server/winston/logger.js
const winston = require('winston');
const winstonDaily = require('winston-daily-rotate-file');
const appRoot = require('app-root-path');
const { createLogger } = require('winston');
const process = require('process'); // 프로그램과 관련된 정보를 나타내는 객체 모듈
const { combine, timestamp, label, printf } = winston.format; // winston.format의 내부 모듈가져오기
const logDir = `${appRoot}/logs`; // logs 디렉토리 하위에 로그 파일저장

// 로그 출력 포맷 정의, printf : 실제 출력양식
const logFormat = printf(({level, message, label, timestamp}) => {
  return `${timestamp} [${label}] ${level}: ${message}`;
  //timestamp : 시간, label : 시스템 네임, level : 정보, 에러 , 경고 인지 구분, message : 개발자가 남긴 로그 메시지
});

// logger 생성
const logger = createLogger({
  format: combine(label({ label: "HotelProject"}), timestamp(),logFormat),
  //포맷종류에는 Simple과 Combine 이 있는데 , winston을 사용할때는 주로 combine을 사용한다
  transports: [
    new winstonDaily({  //log파일 설정
      level: "info", //심각도
      datePattern: "YYYY-MM-DD", //날짜 포맷방식
      dirname: logDir, //디렉토리 파일 이름 설정
      filename: "%DATE%.log",  //파일이름 설정, 자동으로 날짜가 들어옴
      maxSize: "20m",  // 로그파이ㅏㄹ 크기, 정의하지 않으면 데이터가 쌓이고, 지금처럼 제한할경우 초과시 앞의 데이터를 지운다
      maxFiles:"30d", // 최근 30일치 로그파일만 보관
    }),

    new winstonDaily({
      level: "error", //심각도
      datePattern: "YYYY-MM-DD", //날짜 포맷방식
      dirname: logDir + `/error`, //디렉토리 파일 이름 설정
      filename: "%DATE%.log",  //파일이름 설정, 자동으로 날짜가 들어옴
      maxSize: "20m",  // 로그파이ㅏㄹ 크기, 정의하지 않으면 데이터가 쌓이고, 지금처럼 제한할경우 초과시 앞의 데이터를 지운다
      maxFiles:"30d", // 최근 30일치 로그파일만 보관
    })
  ]
})

   //개발 환경일 경우 터미널에서도 로그를 확인할 수 있도록 추가설정 (process 모듈 사용)
   if(process.env.NODE_ENV != "prod"){ //NODE_ENV 환경설정가능, 환경이 prod가 아닌경우 (개발환경일 경우), Process 모듈을 통해 터미널 창에 로그를 확인할 수 있도록설정
    logger.add(
      new winston.transports.Console({
        foramt: winston.format.combine(
          winston.format.colorize(),  //로그 출력시 구분할 수 있도록 색상 추가
          winston.format.simple() // 메시지 형태를 단순하게 설정, Prod 이 아닐경우 폴더와 터미널창에서 로그를 확인 할 수 있도록설정
        )
      })
    )
  }

module.exports = logger;
```

프로젝트 적용시 

```jsx
const logger = require('../../winston/logger');
//...코드

if (error) {
	logger.error("Database error", error);
	return res.status(500).send({ error: "데이터베이스 조회 중 오류가 발생했습니다." });
	}

//... 코드
 try {
        await db.query("INSERT INTO hotels SET ?", newHotel);
        await awsService.getBucketObjects();
        res.status(200).send({ message: "호텔이 성공적으로 등록되었습니다." });
      } catch (err) {
        logger.error("Error on hotel registration", err);
        res.status(500).send({ error: "호텔 등록 중 오류가 발생했습니다." });
      }
    } else {
      res.status(403).send({ error: "hotel_admin만 호텔 등록이 가능합니다." });
    }

```

이런식으로 출력이된다..

![Screenshot 2024-03-07 at 11 57 45 am](https://github.com/henryKyuhyun/Study/assets/118201123/cdf777fb-f6cc-435a-a96e-c6ddbd05ce6e)


```json
{
    "keep": {
        "days": true,
        "amount": 30
    },
    "auditLog": "/Users/kyuhyunpark/Desktop/HotelByKaaPaaa/HotelKP/logs/.d9d067cf872f117d35c58424467b8dfbe782e808-audit.json",
    "files": [
        {
            "date": 1709704209769,
            "name": "/Users/kyuhyunpark/Desktop/HotelByKaaPaaa/HotelKP/logs/2024-03-06.log",
            "hash": "5006e19568ad0f727037b86948a84ffeef1fd5d455e469df6c1bc771c4b51e67"
        }
    ],
    "hashType": "sha256"
}
```

![Screenshot 2024-03-07 at 11 57 55 am](https://github.com/henryKyuhyun/Study/assets/118201123/31d96da8-f32a-4b21-bf9b-b6700110072e)


**결론**

**Winston은 좀 더 전문적이고 유연한 로깅 역량을 제공하고, 모니터링 시스템과 통합하기에 용이합니다. 특히 대규모 애플리케이션이나 복잡한 시스템을 운영할 경우, Winston과 같은 로깅 프레임워크를 사용하여 로그를 관리하는 것이 유리합니다. 로깅 방식이나 선호하는 로깅 구조에 따라 적합한 로깅 라이브러리를 선택하면 됩니다.**

**더 자세한 내용이나 실습 코드가 필요하시다면, [Winston의 공식 문서](https://github.com/winstonjs/winston)(** [https://github.com/winstonjs/winston](https://github.com/winstonjs/winston) **)를 참고하시면됩니다.**
