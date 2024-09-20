# PAM인증 모듈을 통한 비밀번호 정책 설정
</br>

## #0. Target & Contents
### Target
#### - PAM(Pluggable Authentication Modules) 라는 리눅스 시스템에서 사용자의 인증을 담당하는 모듈을 사용하여 </br>  비밀번호를 8자리 이상으로 규제해보자. 

### Contents
1. PAM 인증 모듈
2. 비밀번호 정책 설정
3. Trouble Shooting
</br></br>
## #1. PAM 인증 모듈
### <1. PAM>
  - 사용자를 인증하고 그 사용자의 서비스에 대한 Access(접근)를 제어하는 모듈화된 방법을 일컫는다.
  - PAM을 사용하는 응용프로그램을 재컴파일 없이 구성 파일(conf)을 수정하여 인증 방법을 변경할 수 있다.

### <2. 지시어 및 매커니즘>
   ```
   mechanism [control] path-to-module [argument]
   ```
   지시문을 통해 PAM을 설정할 수도 있는데 위와 같은 형식을 따른다.</br>
   - mechanism : auth, account, password, session)</br>
   - control   :  (include, optional, required)</br>
   - path-to-module : 사용될 모듈</br>
   - argumnet  : 모듈에 적용할 인자 </br>
</br>
  
   ### <Mechanism>
   - auth
      - 요청자의 인증을 처리하고 계정의 권한 설정
      - 계정 설정을 설정. uid, gid, 그룹 및 리소스 제한 설정
   - account
      - 요청된 계정이 사용 가능한지 확인
      - 인증 외의 이유로 계정의 사용 가능성과 관련된다. (ex.시간제한)
   - session
      - 세션 설정과 관련된 작업(ex.로깅)을 수행한다.
      - 세션 종료와 관련된 작업을 수행한다.
    - password
      - 계정과 연결된 인증 토큰(만료 또는 변경)을 수정하는데 사용된다.
    </br></br>
   ### <control : 제어플래그>
    mechanism은 success 또는 failure를 나타내는데, 제어플래그는 PAM에게 이 결과를 처리하는 방법을 알려준다.
    </br>
  
   - required : 모든 필수 모듈의 success가 필요하다.</br>
         - 필수로 표시된 모듈의 확인이 실패하면 해당 인터페이스에 연결된 모든 모듈의 확인이 완료될 때 까지 사용자에게 알리지 않는다.</br>
     </br>
   - requisite : 모든 requisite 모듈을 성공적으로 완료해야 한다.</br>
         - 모듈 실패시 요청이 즉시 거부된다.</br>
     </br>
   - sufficient : 특정 조건 하에서 사용자를 "초기"로 허용하는데 사용할 수 있다.</br>
         - sufficient모듈 실패시 sufficient 모듈이 무시되고, 체인의 나머지 부분이 실행된다. </br>
         - 그러나 required나 requisite로 표시된 모듈이 검사에 실패하면 sufficient 모듈의 성공은 무시되고, 요청이 실패한다.</br>
         </br>
   - optional 
         - 모듈이 실행되지만 요청 결과는 무시된다.</br>
         - 체인의 모든 모듈이 optional로 표시돼있다면 모든 요청이 항상 수락될 것이다.</br>
         </br>

  </br>
  
  ### /etc/pem.d/common-password
  <img width="647" alt="{9900DAC5-C4EF-495B-AA28-DEA2CDBC8A9B}" src="https://github.com/user-attachments/assets/04c00f8e-24ce-4bcc-ad48-98ae2dceea23"></br>
  위 사진을 통해 지시어가 어떻게 적용돼있는지 알 수 있다.
  </br>

#### 3. PAM 모듈
PAM에는 많은 모듈이 있는데 이번에는 pam_cracklib 모듈을 사용한다.
- pam_unix : 전역 인증 정책 관리
- pam_cracklib : 암호 테스트 <- 암호 규제 관리 모듈
</br></br>
## #2. 비밀번호 정책 설정
/etc/pam.d/commom-password에서 모듈을 사용한 지시어로 비밀번호 규제를 하자.</br></br>

### <Trouble Shooting 1>
pam_cracklib.so 모듈을 통해 비밀번호 규제를 관리하려 했으나 현재 환경에서 모듈이 존재하지 않아 pam_pwquality.so 로 대체하였다. 
</br></br>
pam_pwquality.so 모듈을 사용하기 위해선 설치가 필요하다.</br>

```sudo apt-get install libpam-pwquality```
</br>

### <Trouble Shooting 2>
지시어는 순차적으로 실행되며 지시어의 control(제어플래그)에 의해 pam_pwquality.so모듈이 실행되지 않을 수 있다. </br>
따라서 pam_pwquality.so 모듈을 사용하는 지시어를 최상단에 적어야 한다.
</br>

### <pam_pwquality.so 인자>
- `retry=n`: 새 비밀번호의 요청을 n회(기본값은 1회)까지 요구.</br></br>
- `difok=n`: 이전 비밀번호와 다른 문자(n개 이상, 기본값은 10개 이상)의 최소값을 요구. </br>
새 비밀번호의 절반이 이전 비밀번호와 다른 경우 새 비밀번호가 확인된다.</br></br>
- `minlen=n`: 최소한 n+1개의 문자를 가진 비밀번호를 요구. 최소한 6자리 이상으로 지정할 수 없다.</br></br>
- `dcredit=-n`: 적어도 n개의 숫자를 포함하는 비밀번호를 요구.</br></br>
- `ucredit=-n`: 적어도 n개의 대문자를 포함하는 비밀번호를 요구.</br></br>
- `credit=-n`: 적어도 n개의 소문자를 포함하는 비밀번호를 요구.</br></br>
- `ocredit=-n`: 적어도 n개의 특수 문자를 포함하는 비밀번호를 요구.</br></br>
- `dictcheck` : difok와 비슷하게 이전 비밀번호와 비교하여 비슷하다면은 비밀번호 설정이 안된다. dictcheck=0 으로 설정한다면 이전 비밀번호와 비교를 하지 않는다.</br></br>

### <완성된 common-password>
<img width="665" alt="{60A88AAE-49CF-444B-BCDD-417C051C630B}" src="https://github.com/user-attachments/assets/0578ed64-14e6-4a28-9c2a-597d663753b3"></br></br>
- 위 지시어를 해석하면...
</br>
  - 계정과 관련된 인증 (비밀번호)를 변경 하려는데 만약 실패한다면 다음 지시어들 (모듈)은 실행하지 않고 중단한다. </br></br>
  - 그리고, 모듈에서는</br>
    - `retry=3`  -> 비밀번호가 틀려도 다시 설정할 기회를 3번 준다.</br></br>
    - `minlen=8` -> 비밀번호는 8문자 이상이어야 한다.</br></br>
    - `dictcheck=0` -> 이전 비밀번호와 비교하지 않는다.</br></br>
    </br></br>


### <비밀번호 설정 완료>
- 비밀번호 변경을 위해서는 `passwd`를 입력하면 된다.
- <img width="346" alt="{3809C342-20CC-49AA-A54A-D84A769861B7}" src="https://github.com/user-attachments/assets/21ef4fd3-d6c0-46d6-a44c-057383822c1b">

## #3. Trouble Shooting
1.  pam_pwquality.so
2.  common-password의 지시어 순서
