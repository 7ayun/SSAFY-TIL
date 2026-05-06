# [DB] Many to many relationships

---

## 1. 다대다 관계(Many to Many)란

한 테이블의 0개 이상의 레코드가 다른 테이블의 0개 이상의 레코드와 관련된 경우를 말한다. 서로가 서로에 대해 0개 이상의 관계를 가지는 구조로, 영화와 배우처럼 한 영화에 여러 배우가 출연하고, 한 배우가 여러 영화에 출연할 수 있는 관계가 대표적인 예다.

| 관계 유형 | 설명 | 예시 |
|-----------|------|------|
| N:1 (ForeignKey) | N쪽이 1쪽을 외래키로 참조 | 댓글 → 게시글 |
| M:N (ManyToManyField) | 중개 테이블을 통해 양방향 참조 | 환자 ↔ 의사 |

---

## 2. N:1 구조의 한계 — 병원 진료 예약 예시

환자(`Patient`)가 의사(`Doctor`)를 `ForeignKey`로 참조하는 구조로 진료 예약 시스템을 구현하면 다음과 같은 문제가 발생한다.

```python
# hospitals/models.py

class Doctor(models.Model):
    name = models.TextField()
    def __str__(self):
        return f'{self.pk}번 의사 {self.name}'

class Patient(models.Model):
    doctor = models.ForeignKey(Doctor, on_delete=models.CASCADE)
    name = models.TextField()
    def __str__(self):
        return f'{self.pk}번 환자 {self.name}'
```

**한계 상황 1 — 데이터 중복 발생**

`carol`이 두 명의 의사에게 진료를 받으려면 환자 테이블에 같은 이름의 레코드를 두 번 생성할 수밖에 없다.

| id | name | doctor_id |
|----|------|-----------|
| 1  | carol | 1 |
| 2  | duke  | 2 |
| 3  | carol | 2 |  ← 동일 인물인데 중복 생성 |

전체 환자 수를 카운팅할 수도 없고, 나중에 환자 정보가 바뀌면 모든 레코드를 수정해야 하는 문제가 생긴다.

**한계 상황 2 — 동시에 두 의사 지정 불가**

한 `create()` 호출에 여러 외래키 값을 동시에 넣는 건 문법적으로 불가능하다.

```python
# SyntaxError 발생
patient4 = Patient.objects.create(name='carol', doctor=doctor1, doctor2)
```

외래키 컬럼에 `'1, 2'` 형태로 값을 저장하는 것도 **제1정규형 위반**으로 DB 타입상 불가능하다.

---

## 3. 중개 모델(Intermediate Model)

N:1의 한계를 해결하기 위해 **예약이라는 행위 자체를 별도의 테이블로 분리**하는 접근법이다. 두 모델을 직접 연결하는 대신, 중간에서 각각을 외래키로 참조하는 별도 모델을 두어 간접적으로 연결한다.

> **중개 모델** = 다대다 관계에서 두 모델을 연결하는 역할을 하는 특별한 기능을 가진 모델

단순히 "의사 A와 환자 B가 연결됐다"는 사실 외에, **언제 예약했는지, 예약의 상태가 무엇인지** 같은 부가 정보도 함께 저장할 수 있다.

```python
# hospitals/models.py

class Patient(models.Model):  # 외래키 제거
    name = models.TextField()
    def __str__(self):
        return f'{self.pk}번 환자 {self.name}'

class Reservation(models.Model):  # 중개 모델 생성
    doctor = models.ForeignKey(Doctor, on_delete=models.CASCADE)
    patient = models.ForeignKey(Patient, on_delete=models.CASCADE)
    def __str__(self):
        return f'{self.doctor_id}번 의사의 {self.patient_id}번 환자'
```

생성된 `hospitals_reservation` 테이블은 외래키 2개만 가진 중간 테이블이다.

| id | doctor_id | patient_id |
|----|-----------|------------|
| 1  | 1         | 1          |
| 2  | 1         | 2          |

**예약 생성 및 조회 코드 예시**

```python
# 예약 생성 (Reservation 클래스가 주체)
reservation1 = Reservation.objects.create(doctor=doctor1, patient=patient1)

# 의사 → 예약 조회 (역참조)
doctor1.reservation_set.all()

# 환자 → 예약 조회 (역참조)
patient1.reservation_set.all()
```

의사와 환자 모두 `Reservation`에 대해 1:N 관계를 가지므로, 양쪽 모두 **역참조(`_set`)**를 통해 예약 정보를 조회해야 한다.

---

## 4. 중개 모델 방식의 아쉬운 점

현재 코드에서 예약을 만드는 주체는 `Reservation` 클래스다. 즉, 예약이라는 사물이 의사와 환자를 재료로 받아서 생성하는 구조로, **사람 친화적인 표현이 아니다**.

```python
# 현재 방식: 예약이 주체
Reservation.objects.create(doctor=doctor1, patient=patient1)

# 원하는 방식: 환자 또는 의사가 주체
patient1.예약_생성(doctor1)
doctor1.예약_생성(patient1)
```

이 문제를 해결하는 것이 바로 `ManyToManyField`다.

---

## 💡 한 줄 요약

> N:1 구조로는 다수 대 다수 관계를 표현할 때 데이터 중복과 표현 한계가 생기며, 이를 중개 모델을 통해 두 모델을 간접 연결하는 방식으로 해결한다.

---

## ❓ 더 찾아볼 것

- 제1정규형(1NF)이란 무엇인가
- Django ORM의 역참조(reverse accessor) 동작 원리
- 중개 모델과 `ManyToManyField`의 내부 동작 차이
