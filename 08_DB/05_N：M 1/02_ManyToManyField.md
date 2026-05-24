# [DB] ManyToManyField

---

## 1. ManyToManyField란

M:N 관계를 설정하는 Django 모델 필드로, 이 필드를 선언하면 **중개 테이블(중개 모델)이 자동으로 생성**된다. 어느 모델에 작성해도 관계는 동일하게 유지되며, 동일한 관계는 한 번만 저장되어 중복이 발생하지 않는다.

```python
ManyToManyField(to, **options)
```

> **중요**: `ManyToManyField`는 실제 물리적 컬럼을 생성하지 않는다. 두 모델의 테이블에는 변화가 없고, 중개 테이블만 자동으로 별도 생성된다.

---

## 2. 어느 모델에 작성해야 할까

어느 쪽에 두든 관계 자체는 동일하다. 차이는 **참조와 역참조 방향**뿐이다.

- 필드를 가진 쪽 → **참조** (직접 필드 접근)
- 필드가 없는 쪽 → **역참조** (`_set` 매니저 사용)

```python
# 예: Patient 쪽에 ManyToManyField를 둔 경우
class Patient(models.Model):
    doctors = models.ManyToManyField(Doctor)  # 복수형 권장

# 참조 (Patient → Doctor)
patient1.doctors.all()

# 역참조 (Doctor → Patient)
doctor1.patient_set.all()
```

> 필드명은 **복수형**으로 쓰는 것을 권장한다. 다대다 관계는 기본적으로 0개 이상의 다수를 참조하기 때문이며, 코드에서 결과가 복수임을 의미론적으로 드러낼 수 있다.

---

## 3. 중개 테이블 생성 규칙

자동 생성되는 중개 테이블의 이름 규칙:

```
앱이름_모델클래스명_필드명
```

예: `hospitals` 앱의 `Patient` 모델에 `doctors` 필드 → `hospitals_patient_doctors`

이 테이블에는 `patient_id`와 `doctor_id` 두 개의 외래키 컬럼만 존재한다.

---

## 4. add() / remove() 메서드

`ManyToManyField`를 통해 생성된 관계는 `add()`와 `remove()`로 조작한다. 양쪽 모델 모두 동등하게 사용할 수 있다.

```python
# 관계 추가 (.add)
patient1.doctors.add(doctor1)          # 환자가 의사를 예약
doctor1.patient_set.add(patient2)      # 의사가 환자를 등록

# 여러 인스턴스 한 번에 추가
patient.doctors.add(doctor2, doctor3)

# 관계 삭제 (.remove)
patient2.doctors.remove(doctor1)       # 환자가 예약 취소
doctor1.patient_set.remove(patient1)   # 의사가 예약 취소
```

> `.remove()`는 중개 테이블의 관계 레코드만 삭제하며, 대상 객체 자체는 삭제되지 않는다.

---

## 5. 기본 ManyToManyField의 한계와 through 옵션

기본 자동 생성 중개 테이블에는 **외래키 2개만 존재**한다. 예약일, 증상 등 추가 정보가 필요하다면 직접 중개 모델을 정의하고 `through` 옵션으로 등록해야 한다.

```python
class Patient(models.Model):
    doctors = models.ManyToManyField(Doctor, through='Reservation')  # through 등록
    name = models.TextField()

class Reservation(models.Model):
    doctor = models.ForeignKey(Doctor, on_delete=models.CASCADE)
    patient = models.ForeignKey(Patient, on_delete=models.CASCADE)
    symptom = models.TextField()                              # 추가 필드
    reserved_at = models.DateTimeField(auto_now_add=True)    # 추가 필드
```

`through` 옵션을 사용하면 직접 정의한 중개 모델을 통해서도 `add()` / `remove()`를 그대로 사용할 수 있다. 단, `add()` 시 추가 필드 값을 `through_defaults` 인자로 전달해야 한다.

```python
# through_defaults로 추가 필드 값 전달
patient2.doctors.add(doctor1, through_defaults={'symptom': 'flu'})

# 직접 중개 모델을 통한 생성도 가능
reservation1 = Reservation(doctor=doctor1, patient=patient1, symptom='headache')
reservation1.save()
```

**직접 중개 모델에서 삭제할 때는 `.delete()` 사용**

```python
reservation1.delete()              # 중개 모델 인스턴스 직접 삭제
doctor1.patient_set.remove(patient2)  # remove()도 여전히 사용 가능
```

---

## 6. ManyToManyField 대표 인자 3가지

| 인자 | 설명 | 기본값 |
|------|------|--------|
| `related_name` | 역참조 시 사용할 매니저 이름 변경 | `모델명_set` |
| `symmetrical` | 자기 참조 시 대칭 여부 설정 | `True` |
| `through` | 직접 정의한 중개 모델 등록 | - |

**related_name**

```python
class Patient(models.Model):
    doctors = models.ManyToManyField(Doctor, related_name='patients')

# 변경 전
doctor.patient_set.all()

# 변경 후 (이전 이름은 더 이상 사용 불가)
doctor.patients.all()
```

**symmetrical**

동일한 모델 간 자기참조(`'self'`) 관계에서만 사용한다. `True`(기본값)이면 A가 B를 참조할 때 B도 자동으로 A를 참조한다.

```python
class Person(models.Model):
    friends = models.ManyToManyField('self')
    # symmetrical=False → 인스타 팔로우처럼 단방향 가능
```

| 값 | 동작 | 예시 |
|----|------|------|
| `True` | 대칭 (A→B이면 B→A 자동) | 친구 관계 |
| `False` | 비대칭 (A→B여도 B→A 아님) | 팔로우 관계 |

---

## 7. M:N 관계 주요 사항 정리

- M:N으로 맺어진 두 테이블에는 **물리적인 변화가 없다**
- `ManyToManyField`는 **중개 테이블**을 자동으로 생성한다
- N:1은 종속(소유)의 관계지만, M:N은 **동등한 양방향 관계**다
- 양쪽 어디서든 `add()`, `remove()`로 관계를 조작할 수 있다

---

## 💡 한 줄 요약

> `ManyToManyField`는 중개 테이블을 자동 생성하여 양쪽 모델이 동등하게 관계를 추가/삭제할 수 있게 해주며, 추가 데이터가 필요할 때는 `through` 옵션으로 직접 중개 모델을 정의한다.

---

## ❓ 더 찾아볼 것

- `ManyToManyField`의 `through_defaults` 인자 상세
- `through` 옵션 사용 시 `add()` 제약 조건
- Django 공식 문서의 추가 ManyToManyField 옵션 (`db_table`, `limit_choices_to` 등)
