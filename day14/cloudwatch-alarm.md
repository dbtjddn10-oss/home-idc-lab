# Day 14 - CloudWatch CPU 경보 및 SNS 알림 테스트

## 1. 목적

AWS EC2 인스턴스의 CPU 사용률을 CloudWatch로 감시하고, 설정한 임계값을 초과했을 때 SNS 이메일 알림이 전달되는지 검증했다.

## 2. 모니터링 구성

```text
EC2 CPUUtilization
    ↓
CloudWatch Alarm
    ↓
Amazon SNS Topic
    ↓
Email Notification
```

## 3. CloudWatch 경보 설정

| 항목 | 설정 |
|---|---|
| 네임스페이스 | AWS/EC2 |
| 지표 | CPUUtilization |
| 통계 | Average |
| 기간 | 5분 |
| 평가 기간 | 1회 |
| 조건 | 1% 이상 |
| 경보 작업 | SNS 이메일 전송 |

임계값 1%는 짧은 실습 시간 안에 경보 동작을 확인하기 위한 테스트 설정이다.

실제 운영 환경에서는 평상시 CPU 사용률, 서비스 중요도와 순간적인 사용량 증가를 고려하여 더 높은 임계값과 여러 번의 연속 평가 기간을 사용해야 한다.

## 4. SNS 알림 설정

CloudWatch 경보의 알림 대상으로 다음 SNS 주제를 사용했다.

```text
home-idc-ec2-alerts
```

이메일 구독을 생성하고 수신된 확인 메일을 통해 구독을 활성화했다.

개인 이메일 주소, AWS 계정 ID, 인스턴스 ID와 ARN 같은 식별 정보는 문서에 기록하지 않았다.

## 5. CPU 부하 테스트

EC2 서버에서 제한된 시간 동안 CPU 부하를 발생시켰다.

```bash
nohup timeout 420s bash -c 'while :; do :; done' >/dev/null 2>&1 &
```

부하 발생 후 EC2의 평균 CPU 사용률이 설정한 임계값을 초과했다.

## 6. 경보 결과

CloudWatch 경보 상태가 다음과 같이 변경됐다.

```text
OK → ALARM
```

이후 SNS를 통해 CloudWatch 경보 이메일이 정상적으로 수신됐다.

검증된 전체 흐름은 다음과 같다.

```text
CPU 부하 발생
    ↓
CloudWatch 지표 수집
    ↓
CPU 임계값 초과
    ↓
ALARM 상태 전환
    ↓
SNS 이메일 전송
```

## 7. 테스트 종료 및 자원 정리

테스트 완료 후 CPU 부하 프로세스를 종료하고 다음 AWS 자원을 정리했다.

- EC2 인스턴스 종료 및 삭제
- EBS 볼륨 삭제 확인
- CloudWatch 경보 삭제
- SNS 주제 및 구독 삭제
- EC2 키 페어 삭제
- 로컬 PEM 개인 키 삭제
- 실습용 보안 그룹 삭제
- 탄력적 IP가 없음을 확인
- NAT 게이트웨이가 없음을 확인

기본 VPC와 기본 보안 그룹은 삭제하지 않았다.

## 8. 결과

EC2 서버 상태를 CloudWatch 지표로 감시하고, 장애 조건을 인위적으로 발생시켜 SNS 이메일 알림까지 전달되는 전체 모니터링 흐름을 직접 구성하고 검증했다.
