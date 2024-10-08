# 📂 스텝 2: 다중 파일 업로드 로직 디버깅 실습 가이드

# 🎯 학습 목표

다중 파일 업로드 로직의 버그를 발견하고 수정하는 과정을 경험합니다.

# 📝 실습 가이드

## 1. 실습 시나리오

### 🪲 버그 리포트

당신은 신입 백엔드 개발자입니다. 당신의 입사 동기인 신입 프론트엔드 개발자로부터 파일 업로드 API에 뭔가 문제가 있다는 메시지를 받았습니다. 얼마전 퇴사한 동료가 만들어둔 API로, 당신은 아직 코드도 못 읽어본 상태입니다.

당신 동기의 말에 따르면 A 유저의 파일들은 업로드가 다 되는데, B 유저의 파일들은 업로드시 일부가 성공했다고 메시지는 나오지만 실제로는 아무것도 업로드가 되지 않는다고 합니다.

동기가 A 유저의 파일들과 B 유저의 파일들을 전달해주었습니다. 퇴사한 동료가 만들어둔 버그를 대신 고쳐봅시다.

### API 목록

- `POST /upload` : "files" key에 여러 파일을 formData 로 전송해서 업로드하고, 업로드 성공한 파일 목록을 응답으로 반환한다.
- `GET /files` : 서버에 업로드된 파일 목록을 반환한다.

## 2. 디버깅 마인드셋 템플릿을 활용한 문제 해결

### Step 1: 문제 정의

현재 풀고자 하는 문제를 한 문장으로 명확하게 정의해보세요.

- 현재 특정 사용자만 파일을 업로드할 수 있지만, 이를 수정하여 모든 사용자가 파일을 업로드할 수 있어야 하고 업로드에 실패하면 실패 메시지를 노출한다.

---

- 특정한 파일 목록을 업로드하려고 할 때, 실제로는 업로드가 안 됐는데, 메시지는 업로드가 되었다고 나오는 문제를 해결한다.
- 어떤 파일이 업로드가 안 되는건지 패턴을 파악한다.
- 패턴 파악을 하기 위해, 동료가 전해준 파일을 가지고 재현 실험을 해본다.
- 패턴 파악이 됐으면, 의사결정
  - 만약 이 패턴이 업로드가 성공하는 것이 정상이면, 업로드 로직을 변경한다.
  - 만약 이 패턴이 업로드가 실패하는 것이 정상이면, 검사하고 메시지를 반환하는 로직을 변경한다.
  - 이 API가 internal API라면, 실패 메시지가 최대한 자세히 나와야 할 것 같다.
    - 어떤 파일이 어떤 이유로 업로드가 안 됐다.
  - public API라면, 실패 메시지가 너무 자세히 나오면 안 될 것 같다.
    - 이 때는 그냥 실패했다. 라고만 나오되
    - 에러 코드 정도는 반환해도 될 거 같다.
    - API document 는 그 재현에 대해 업데이트해야 할 것.

### Step 2: 올바른 동작 정의

파일 업로드 기능이 올바르게 동작한다면 어떤 일이 벌어져야 하는지 정의해보세요. 이를 명확하게 정의하기 위해 올바른 동작을 given, when, then 형식으로 기술해보세요.

- given: 사용자가 파일을 선택하여
- when: 서버에 전송했을 때
- then: 업로드가 성공했을 때만 성공 메시지를 노출하고, 실패했을 땐 실패 메시지를 노출한다.

---

1. given: 유저가 파일을 하나 선택해서 올렸다.
2. when: 그 파일이 API가 허용하는 패턴이 아니다.
3. then: 업로드가 실제로 실패하고, 실패 메시지가 떠야 한다. -> 실패 메시지는 이 파일이 어떤 이유로 업로드가 실패했는지 기술되어야 한다.

4. given: 유저가 파일을 하나 선택해서 올렸다.
5. when: 그 파일이 API가 허용하는 패턴이다.
6. then: 업로드가 실제로 성공하고, 성공 메시지가 떠야 한다.

7. given: 유저가 파일을 2개 이상 선택해서 올렸다.
8. when: 모든 파일이 API가 허용하는 패턴이다.
9. then: 업로드가 실제로 성공하고, 성공 메시지가 떠야 한다.

10. given: 유저가 파일을 2개 이상 선택해서 올렸다.
11. when: 하나의 파일이라도 API가 허용하는 패턴이 아니다.
12. then: 업로드가 실제로 실패하고, 실패 메시지가 떠야 한다. -> 실패 메시지는 어떤 파일이 어떤 이유로 업로드가 실패했는지 기술되어야 한다.

### Step 3: 최소 재현 환경 구축하며 관찰

문제가 발생한 시점을 정확히 확인 후 내 환경에서 직접 재현하기 위해 문제가 있는 부분을 격리시켜보세요. 그리고 재현 환경과 정상 동작의 차이점을 관찰하고, 관찰한 내용을 적어보세요.

- files API 호출해서 어떤 파일들이 있는지 확인
  - 유저 A 파일 선택해서 업로드 후 GET API 호출하여 관찰
  - 유저 B 파일 선택해서 업로드 후 GET API 호출하여 관찰

### Step 4: 차이를 발생시키는 다양한 원인 탐색

Step 3에서 관찰한 차이를 발생시키는 원인이 될 만한 옵션들을 자유롭게 적어보세요. 옵션을 적기 어렵다면 추가 정보를 수집해보세요.

- 사용자가 서버에서 허용하지 않은 파일 확장자를 업로드하여 실제로는 업로드가 되지 않았다.
- 실제 업로드 결과에 관계없이 업로드 성공 메시지를 노출한다. (-> 서버 업로드에 실패했을 때 에러 처리 로직이 없어서 무조건 성공 메시지를 노출하고 있었다.)
- 서버에서는 파일 확장자에 관계없이 업로드를 허용하지만 실제로는 잘못된 확장자여서 이를 프론트에서 렌더링하지 못했다.

---

- svg 확장자는 허용이 안 된다.
- 파일 용량이 너무 크다.
- 파일 이름이 너무 길다. -> 이름 같아서 아님
- 파일 이름이 서버에 존재하는 파일과 중복이다. -> 아님, b2는 없었음
- 한 번에 올릴 수 있는 파일 개수를 넘음 -> 똑같이 2개 해도 안 됐고, 1개만 해도 안 됐음

### Step 5: 가설 설정 및 검증

Step 4에서 도출한 옵션 중 하나를 선택하여 검증 가능한 가설을 세워보세요. 그리고 코드에 작은 변경을 가하면서 가설대로 현상이 변하는지 관찰해보세요.

만약 가설이 틀렸다면 그 이유를 적어보고, 다른 가설로 넘어가 반복해보세요.

가설 문장

- svg 확장자가 허용이 안 된다면, 파일 확장자를 변경했을 때 업로드가 되어야 한다.

  - B 유저가 업로드 한 각각의 파일을 서로 확장자를 바꿔서 테스트한다.
  - svg 파일도 업로드 할 수 있다. 해당 문제가 아님.

- 파일 용량이 문제라면, 성공한 파일과 실패한 파일의 크기가 다를 것.
  - svg 파일만 3메가임
  - limits 가 2메가임
  - 파일 업로드 크기 제한에 걸림

<details>
<summary>힌트</summary>

- 파일 업로드 시 발생하는 오류를 처리하고, 사용자에게 에러 메시지를 반환해야 합니다.
- 예를 들어, 파일 크기 제한을 초과한 파일이 있을 때 각 파일의 업로드 상태를 기록하고, 에러 메시지를 포함하여 사용자에게 반환할 수 있습니다.

간단한 에러 처리 예시:

```jsx
app.post("/upload", (req, res) => {
  upload(req, res, (err) => {
    if (err) {
      return res.status(400).json({ message: err.message });
    }

    const uploadedFiles = req.files.map((file) => ({
      fileName: file.originalname,
      filePath: file.path,
      fileSize: file.size,
    }));

    res.json({
      message: "Files uploaded successfully.",
      files: uploadedFiles,
    });
  });
});
```

console을 이용하여 `err.code`를 출력하여 어떤 에러가 발생했는지 확인하고 이를 처리하는 방법을 생각해보세요.

</details>
