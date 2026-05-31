---
title: "[React] React Hook Form으로 복잡한 폼 상태 관리하기"
date: 2026-05-28T17:30:00Z
categories: [frontend]
tags: [react, react-hook-form, form, typescript, frontend]
description: "회원가입 폼과 캐릭터 생성 폼에서 React Hook Form을 활용해 입력값, 동적 배열, 커스텀 UI, 서버 에러, 페이지 이탈 방지까지 하나의 흐름으로 관리한 경험을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 폼은 생각보다 많은 책임을 가져요

프론트엔드에서 폼은 단순한 `input` 모음처럼 보이지만, 실제 서비스에서는 꽤 많은 책임을 가집니다.

입력값 저장, 유효성 상태, 에러 메시지, 파일 업로드, 동적 리스트, 미리보기, 페이지 이탈 방지까지 모두 폼과 연결됩니다. 필드가 몇 개 없을 때는 `useState`만으로도 충분하지만, 폼이 커지는 순간부터 상태가 여러 컴포넌트에 흩어지고 관리 포인트가 빠르게 늘어납니다.

이번 프로젝트에서는 `react-hook-form`을 회원가입 폼과 캐릭터 생성 폼에 사용했습니다. 특히 캐릭터 생성 화면은 입력 필드가 많고, 에셋과 시나리오처럼 배열 형태의 데이터도 포함하고 있어서 폼 상태가 쉽게 복잡해질 수 있었어요.

이 글에서는 `zod` 같은 스키마 검증 도구는 잠시 제외하고, **React Hook Form 자체를 어떻게 활용했는지**를 중심으로 정리해보겠습니다. 폼 값이 어디에 저장되고, 어떤 방식으로 하위 컴포넌트와 연결되는지부터 차근차근 살펴볼게요.

---

## 💡 왜 React Hook Form을 선택했을까요?

처음 폼을 구현할 때 가장 단순한 방법은 각 필드를 `useState`로 관리하는 것입니다.

하지만 필드가 많아지고, 여러 컴포넌트에서 같은 폼 값을 읽거나 수정해야 하는 순간부터 복잡도가 빠르게 올라갑니다. 예를 들어 캐릭터 생성 폼에는 다음과 같은 값들이 있습니다.

```ts
interface CharacterCreateFormValues {
  representativeImage: string;
  title: string;
  name: string;
  characterIntroduce: string;
  characterDetailSetting: string;
  asset?: CharacterAsset[];
  scenarios: ScenarioItem[];
  tagIds: { id: number; label: string }[];
  isPublic: boolean;
  characterDescription: string;
  tendency: string;
  category: string;
}
```

문자열 필드뿐 아니라 이미지, 에셋 배열, 시나리오 배열, 태그 배열까지 포함되어 있습니다.

이 구조를 전부 `useState`로 관리하면 값 변경 함수, 검증 상태, 초기화 로직, 저장 로직이 여러 곳에 흩어지기 쉽습니다. React Hook Form을 사용하면 폼 상태는 하나의 흐름으로 관리하면서도, UI 컴포넌트는 작게 분리할 수 있습니다.

---

## 🛠️ FormProvider로 props drilling 줄이기

이 프로젝트에서는 폼의 최상위 컴포넌트에서 `useForm`을 선언하고, `FormProvider`로 하위 컴포넌트에 폼 메서드를 공유했습니다.

회원가입 페이지의 구조는 다음과 같습니다.

```tsx
const methods = useForm<AuthFormValues>({
  mode: "onChange",
  reValidateMode: "onChange",
  defaultValues: {
    nickname: "",
    email: "",
    code: "",
    password: "",
    passwordCheck: "",
    signupToken: "",
    isPrivacyAgreed: false,
    isTermsAgreed: false,
  },
});

return (
  <FormProvider {...methods}>
    <SignupForm />
  </FormProvider>
);
```

이렇게 하면 `SignupForm` 아래에 있는 `NicknameField`, `PasswordField`, `PasswordCheckField`, `Agreed` 같은 컴포넌트에서 `register`, `control`, `errors`를 계속 props로 내려보낼 필요가 없습니다.

하위 컴포넌트에서는 `useFormContext`로 필요한 값만 꺼내면 됩니다.

```tsx
const {
  register,
  control,
  formState: { errors },
} = useFormContext();
```

덕분에 각 필드 컴포넌트는 자신의 UI와 입력 규칙에만 집중할 수 있었습니다.

---

## ✍️ register로 기본 input 연결하기

기본적인 `input`은 `register`로 React Hook Form에 연결합니다.

닉네임 필드는 다음처럼 작성했습니다.

```tsx
<SmartInput
  {...register("nickname", {
    required: "닉네임을 입력해주세요.",
    maxLength: {
      value: 20,
      message: "최대 20자까지 가능합니다.",
    },
    pattern: {
      value: NICKNAME_REGEX,
      message: "특수문자는 사용할 수 없습니다.",
    },
  })}
  label="닉네임"
  required
  value={nicknameValue}
  maxLength={20}
  error={error?.message as string}
/>
```

`register`를 사용하면 입력값 추적, `blur`/`change` 이벤트 처리, 기본 검증 규칙 연결을 React Hook Form이 담당합니다.

여기서 중요한 점은 필드 컴포넌트가 폼 전체 구조를 알 필요가 없다는 것입니다. `NicknameField`는 오직 `nickname`이라는 필드의 입력과 에러 표시에 집중하면 됩니다.

---

## 👀 useWatch로 필요한 값만 구독하기

폼 화면에서는 특정 값이 바뀔 때 UI도 함께 바뀌어야 하는 경우가 많습니다. 회원가입 버튼 활성화, 닉네임 중복 확인, 캐릭터 미리보기 같은 기능이 대표적이에요.

이 프로젝트에서는 `useWatch`를 사용해 필요한 값만 구독했습니다.

회원가입 폼에서는 필수 입력값과 약관 동의 여부를 구독해 버튼 활성화 여부를 계산했습니다.

```tsx
const {
  nickname = "",
  email = "",
  password = "",
  passwordCheck = "",
  isPrivacyAgreed = "",
  isTermsAgreed = "",
} = useWatch({ control });

const isFormValid =
  !!(
    nickname &&
    email &&
    password &&
    passwordCheck &&
    isPrivacyAgreed &&
    isTermsAgreed
  ) &&
  Object.keys(errors).length === 0 &&
  password === passwordCheck;
```

닉네임 필드에서는 현재 입력값을 구독하고, `debounce`를 적용한 뒤 중복 확인 API를 호출했습니다.

```tsx
const nicknameValue = useWatch({ control, name: "nickname" });

const debouncedNickname = useDebounce({
  value: nicknameValue,
  delay: 500,
});
```

이렇게 하면 사용자가 입력할 때마다 바로 API를 호출하지 않고, 일정 시간 입력이 멈춘 뒤에만 중복 확인을 수행할 수 있습니다.

캐릭터 생성 화면에서는 `useWatch`가 미리보기 기능과 연결됩니다.

```tsx
const scenarios = useWatch({
  control,
  name: "scenarios",
});

const name = useWatch({
  control,
  name: "name",
});
```

사용자가 캐릭터 이름이나 시나리오 내용을 수정하면 오른쪽 미리보기 화면도 자연스럽게 업데이트됩니다.

---

## 🧩 useFieldArray로 동적 리스트 관리하기

캐릭터 생성 폼에서 가장 복잡한 부분은 배열 데이터였습니다. 사용자는 에셋을 여러 개 추가할 수 있고, 시나리오도 여러 개 만들 수 있습니다.

이런 동적 배열 필드는 `useFieldArray`로 관리했습니다.

```tsx
const { fields, append, remove, move } = useFieldArray({
  control,
  name: "asset",
});
```

여기서 `fields`, `append`, `remove`, `move`는 배열 필드를 다룰 때 핵심이 되는 값과 함수입니다.

- `fields`: 현재 배열 필드의 목록입니다. React Hook Form이 각 항목에 고유한 `id`를 붙여주기 때문에 렌더링할 때 `key`로 사용할 수 있습니다.
- `append`: 배열의 마지막에 새 항목을 추가합니다. 사용자가 파일을 업로드하거나 새 시나리오를 만들 때 사용합니다.
- `remove`: 특정 인덱스의 항목을 배열에서 제거합니다. 삭제 버튼을 눌렀을 때 사용합니다.
- `move`: 배열 안에서 항목의 위치를 바꿉니다. 드래그 앤 드롭으로 순서를 변경할 때 유용합니다.

에셋을 추가할 때는 파일을 선택하고, `FileReader`로 미리보기용 이미지를 만든 뒤 배열에 새 항목을 추가했습니다.

```tsx
append({
  assetFile: null,
  assetName: file.name.split(".").slice(0, -1).join("."),
  assetImage: reader.result as string,
  assetSituation: "",
});
```

삭제할 때는 `remove(index)`처럼 제거할 항목의 인덱스를 넘기면 됩니다. 순서를 변경할 때는 `move(from, to)` 형태로 기존 위치와 이동할 위치를 넘깁니다.

```tsx
const onDragEnd = (result: DropResult) => {
  if (!result.destination) return;

  move(result.source.index, result.destination.index);
};
```

직접 배열을 복사해서 `setState` 하는 방식보다 훨씬 안정적입니다. React Hook Form이 각 필드의 `id`와 상태를 관리해주기 때문에, 동적 리스트에서 흔히 생기는 렌더링 문제도 줄일 수 있습니다.

시나리오도 같은 방식으로 관리했습니다.

```tsx
const { fields, append, remove } = useFieldArray({
  control,
  name: "scenarios",
});

append({
  name: "다른 시나리오",
  contents: [],
});
```

이처럼 배열 구조가 깊어져도 `scenarios.0.name`, `scenarios.1.contents` 같은 경로 기반 접근으로 폼 데이터를 다룰 수 있습니다.

---

## 🔗 setValue와 getValues로 커스텀 UI 연결하기

모든 입력이 단순 `input`으로 끝나지는 않습니다. 파일 업로드, 채팅 미리보기, 토글, 태그 선택처럼 커스텀 UI에서 폼 값을 직접 변경해야 하는 경우가 있습니다.

이럴 때는 `setValue`를 사용합니다.

프로필 이미지 필드에서는 파일 객체와 미리보기 이미지를 각각 폼 상태에 저장했습니다.

```tsx
setValue("profileImgFile", file, {
  shouldValidate: true,
});

setValue(name, reader.result, {
  shouldValidate: true,
});
```

캐릭터 미리보기에서는 사용자가 입력한 메시지를 현재 시나리오의 `contents` 배열에 추가했습니다.

```tsx
const currentContents =
  getValues(`scenarios.${activeScenarioIndex}.contents`) || [];

setValue(
  `scenarios.${activeScenarioIndex}.contents`,
  [...currentContents, newContent],
  { shouldValidate: true },
);
```

여기서 `getValues`는 현재 폼 값을 읽는 용도로 사용하고, `setValue`는 특정 필드 경로의 값을 갱신하는 용도로 사용합니다.

이 방식 덕분에 복잡한 UI도 폼 상태와 분리되지 않습니다. 화면에서는 채팅을 추가하는 것처럼 보이지만, 내부적으로는 React Hook Form의 `scenarios[n].contents` 값이 업데이트됩니다.

---

## 🚨 서버 에러를 폼 에러로 통합하기

클라이언트에서 기본 검증을 통과해도 서버에서 에러가 발생할 수 있습니다. 예를 들어 이미 사용 중인 이메일이나 닉네임 같은 경우입니다.

이 프로젝트에서는 서버에서 내려온 필드별 에러를 React Hook Form의 `setError`로 연결하는 커스텀 훅을 만들었습니다.

```ts
export const useFormServerError = <T extends FieldValues>() => {
  const { setError, setFocus } = useFormContext<T>();

  const setFieldErrors = (fields?: Partial<Record<string, string>>) => {
    if (!fields) return false;

    const entries = Object.entries(fields);
    const lastErrorKey = entries[entries.length - 1][0];

    setFocus(lastErrorKey as Path<T>);

    entries.forEach(([key, message]) => {
      setError(key as Path<T>, {
        type: "server",
        message,
      });
    });

    return true;
  };

  return { setFieldErrors };
};
```

회원가입 요청 실패 시에는 서버 에러를 그대로 폼 에러로 반영했습니다.

```tsx
authRegister(data, {
  onError: (error) => {
    setFieldErrors(error.fields);
  },
});
```

이렇게 하면 클라이언트 에러와 서버 에러를 같은 방식으로 UI에 표시할 수 있습니다. 사용자는 어떤 검증 단계에서 발생한 에러인지 몰라도 해당 필드에서 자연스럽게 에러 메시지를 확인할 수 있습니다.

---

## 💾 isDirty로 임시 저장과 이탈 방지 처리하기

캐릭터 생성처럼 작성 시간이 긴 폼에서는 사용자가 실수로 페이지를 떠나는 상황을 막아야 합니다.

React Hook Form은 `formState.isDirty`를 통해 폼이 초기값에서 변경되었는지 알려줍니다.

```tsx
const {
  formState: { isDirty },
  reset,
  getValues,
} = methods;
```

임시 저장 후에는 현재 값을 기준으로 `reset`을 호출했습니다.

```tsx
const handleSave = async () => {
  const currentData = getValues();

  reset(currentData);
  alert("임시저장이 완료되었습니다.");
};
```

이렇게 하면 저장 이후에는 현재 상태가 새로운 기준점이 되기 때문에 `isDirty`가 다시 `false`가 됩니다.

페이지 이탈 방지 로직에서도 `isDirty`를 사용했습니다.

```tsx
useNavigationGuard({
  enabled: isDirty,
  confirm: (info) => {
    setPendingPath(info.to);
    setActiveModal("UNSAVED");
    return false;
  },
});
```

폼 변경 여부를 직접 비교하지 않고도 React Hook Form의 상태를 활용해 UX를 구현할 수 있었습니다.

---

## ✅ 정리

이 프로젝트에서 React Hook Form은 폼 상태 관리를 단순화하는 도구 이상이었습니다. 여러 컴포넌트로 나뉜 폼을 하나의 흐름으로 묶고, 복잡한 사용자 입력을 안정적으로 다루는 기반이 되어주었습니다.

핵심적으로 사용한 방식은 다음과 같습니다.

- `FormProvider`로 폼 메서드를 하위 컴포넌트에 공유
- `useFormContext`로 props drilling 없이 필드 컴포넌트에서 폼 접근
- `register`로 기본 input과 검증 규칙 연결
- `useWatch`로 필요한 값만 구독해 버튼 활성화, 중복 확인, 미리보기 구현
- `useFieldArray`로 에셋과 시나리오 같은 동적 배열 관리
- `setValue`, `getValues`로 파일 업로드나 채팅형 입력 같은 커스텀 UI 연결
- `setError`, `setFocus`로 서버 에러를 폼 에러 흐름에 통합
- `isDirty`, `reset`으로 임시 저장과 페이지 이탈 방지 처리

폼이 단순할 때는 React Hook Form의 장점이 크게 체감되지 않을 수 있습니다. 하지만 입력 필드가 많아지고, 컴포넌트가 분리되고, 배열 데이터와 커스텀 UI가 들어가기 시작하면 장점이 뚜렷해집니다.

결국 이 프로젝트에서 React Hook Form을 사용한 가장 큰 이유는 **폼 상태를 한곳에서 관리하면서도, UI는 작고 독립적인 컴포넌트로 유지할 수 있었기 때문**입니다.
