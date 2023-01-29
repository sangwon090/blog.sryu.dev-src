---
title: "한컴타자연습 이스터에그 분석"
date: 2023-01-19T18:46:56+09:00
---

고등학교 2학년 때 한컴타자연습 리소스에 이스터에그가 숨겨져 있다는 사실을 우연히 알게 됐는데요, 이스터에그가
더미 데이터로만 존재하는지, 아니면 프로그램 상에서 띄우는 방법이 따로 있는지 궁금해서 한컴타자연습을 리버싱 했었습니다.
프로그램 구조가 많이 단순해 이스터에그를 띄우는 키 조합을 금방 알아낼 수 있었고, 리버싱 결과를 페이스북의
[코딩이랑 무관합니다만 그룹](https://www.facebook.com/groups/System.out.Coding/permalink/4095674740492190/)에 올려놨습니다.

![페이스북 스크린샷](/images/hancom-typing-training-easter-egg/comu.png)

블로그를 개설한 지 한 달 정도 됐는데, 뭐라도 올려야 될 것 같아서 키 조합을 어떻게 알아냈는지 리버싱 과정을 정리해서
올려볼려고 합니다. 2년 전에 리버싱 과정을 따로 기록해두지 않아 이번에 리버싱을 다시 했는데, 재작년에 발견하지 못한
또 다른 키 조합도 발견했네요.

## 시작하기 전에
이 글은 한컴타자연습 2005를 기준으로 작성되었습니다. 2002, 2004 버전의 경우 아마도 큰 차이점은 없을텐데 함수의 이름이ㅇ
다를겁니다. 참고로 2007 버전에서는 이스터에그 관련 코드가 제거되었습니다.

프로그램이 단순해서 리버싱을 제대로 공부해본 적이 없었지만, IDA의 디컴파일러만으로 이스터에그를 쉽게 찾을 수 있었습니다.
리버싱에 관심이 있으신 분은 제 글을 읽기 전에 한번 직접 해보는 것도 나쁘지 않을 것 같습니다.

## 리소스 훑어보기
어떤 이스터에그가 있는지 한 번 살펴볼까요?

1. 첫 번째 이스터에그 `169 (0xA9)` ![이스터에그 리소스 1](/images/hancom-typing-training-easter-egg/resource_hacker_01.png#center)
2. 두 번째 이스터에그 `346 (0x15A)` ![이스터에그 리소스 2](/images/hancom-typing-training-easter-egg/resource_hacker_02.png#center)

프로그램 개발이 수능에까지 영향을 미쳤다는 부분이 인상적이네요.

## 이스터에그를 표시하는 코드

### LoadImageA
프로그램 전체를 분석하는 것 보다 이스터에그와 연관된 부분만 분석하는게 훨씬 효율적이겠죠? 이스터에그를 화면에 띄우려면
이미지를 불러오는 함수를 호출할 수 밖에 없을 것입니다. 그러므로 `LoadImageA` 함수의 크로스 레퍼런스를 분석하는 것에서
시작해봅시다.

![LoadImageA xref 목록](/images/hancom-typing-training-easter-egg/LoadImageA%20xref.png#center)

4개의 xref를 가지고 있네요. 하나씩 확인해봅시다.

---

`sub_4069A0`의 디컴파일 결과입니다.

```C
HANDLE __cdecl sub_4069A0(LPCSTR name, _DWORD *a2)
{
    HANDLE result; // eax
    CHAR Text[256]; // [esp+8h] [ebp-100h] BYREF

    *a2 = 0;
    result = LoadImageA(hInst, name, 0, 0, 0, 0x10u);
    *a2 = result;
    if ( !result )
    {
        sprintf(Text, aS_0, name);
        return (HANDLE)MessageBoxA(0, Text, aLoad, 0);
    }
    return result;
}
```

`LoadImageA`의 5번째 인자로 `LR_LOADFROMFILE (0x10)`을 넘기네요. `name` 으로 넘어온 이름을 가진 파일을 로드하고 리턴하는
함수로 보입니다. 이스터에그 이미지가 리소스에 있을 뿐더러, `sub_4096A0`의 xref가 `WinMain+22F`에 있는
`sub_4069A0(aHnctlogoSys, &dword_932120);`밖에 없는 것으로 봤을 때 로고를 로드하는 함수인 것으로 보입니다. 패스하겠습니다.

---

`sub_406A10`의 디컴파일 결과입니다.

```C
HANDLE __cdecl sub_406A10(LPCSTR name, _DWORD *a2)
{
    HANDLE result; // eax
    CHAR Text[256]; // [esp+8h] [ebp-100h] BYREF

    *a2 = 0;
    result = LoadImageA(hInst, name, 0, 0, 0, 0x2000u);
    *a2 = result;
    if ( !result )
    {
        sprintf(Text, aS_0, name);
        return (HANDLE)MessageBoxA(0, Text, aLoad, 0);
    }
    return result;
}
```

`sub_406910`과 거의 똑같네요. 딱 한가지만 다릅니다. `LoadImageA`의 5번째 인자로 `LR_CREATEDIBSECTION (0x2000)`를 넘깁니다.
xref가 172개나 되네요. `LoadImageA`의 xref 2개를 더 분석하고, 이스터에그와 연관이 없다고 보이면 그 때 다시 분석하겠습니다.

---

`WinMain+BA`의 디컴파일 결과입니다.

```C
v18.hIconSm = (HICON)LoadImageA(hInst, name, 1u, 16, 16, 0);
```

아이콘을 불러오는 코드입니다. 패스합니다.

---

`WinMain+`의 디컴파일 결과입니다.

```C
v19.hIconSm = (HICON)LoadImageA(hInst, name, 1u, 16, 16, 0);
```

이것도 아이콘을 불러오는 코드입니다. 되돌아가서 `sub_406A10`의 xref를 분석해봅시다.

### sub_406A10

![sub_406A10의 xref](/images/hancom-typing-training-easter-egg/sub_406A10%20xref.png#center)

xref가 172개나 돼서 앞이 캄캄했는데, 모두 `sub_41CF80`이라는 함수에서 호출되네요. 다행입니다.

### sub_41CF80

```C
HANDLE sub_41CF80()
{
    sub_406A10((LPCSTR)0x14C, &dword_C278E4);
    sub_406A10((LPCSTR)0x75, &dword_C2796C);
    sub_406A10((LPCSTR)0xAF, &dword_C0E8C4);
    // ...
    sub_406A10((LPCSTR)0xA9, &dword_C29B40);
    sub_406A10((LPCSTR)0x15A, &dword_C278E0);
    // ...
    sub_406A10((LPCSTR)0x14B, &dword_9305F4);
    sub_406A10((LPCSTR)0x153, &dword_45B37C);
    return sub_406A10((LPCSTR)0x158, &dword_461E6C);
}
```

아까 리소스해커로 봤듯이, 이스터에그의 리소스 id가 `169 (0xA9)`와 `346 (0x15A)`였습니다.
리소스가 `sub_406A10` 함수로 로드되어 `dword_C29B40`과 `dword_C278E0`에 저장되는 것 같습니다.
이 주소에 접근하는 코드를 분석하면 이스터에그 발동 조건을 알아낼 수 있을 것입니다.

### dword_C29B40

![dword_C29B40의 xref](/images/hancom-typing-training-easter-egg/dword_C29B40%20xref.png#center)

첫 번째 xref는 방금 분석했던 함수이고, 두 번째 xref는 `sub_41D9D0`에 있는 `DeleteObject(dword_C29B40);`입니다.
세 번째, 네 번째 xref는 `sub_41F5D0`에 있고, 둘 다 이스터에그 이미지를 띄우는 부분으로 추정됩니다.

## 이스터에그를 발동시키는 로직
### sub_41F5D0 (1)

필요 없는 부분은 생략했습니다.

```C
v3 = GetAsyncKeyState;  // <-- v3을 주목해주세요.
while ( 1 )
{
    // ...
    if ( (int (*)())dword_A570EC != sub_41CC30 )
    {
        dword_A570EC = (int)sub_41CC30;
        sub_41CE50();
    }
    v4 = sub_41F340();  // <-- v4를 주목해주세요.
    String[0] = 0;
    sub_4097F0(-100, -100, 0, (wchar_t *)String, 0);
    if ( v4 == 8 && v3(17) && v3(16) )
        ++v41;
    if ( v41 <= 11 )
        goto LABEL_50;
    if ( v4 == 73 )
        ++v40;
    if ( v40 > 0 )
    {
        if ( v4 == 78 )
            ++v38;
        if ( v38 > 1 && v4 == 79 )
            ++v39;
    }
    if ( v39 <= 0 )
        goto LABEL_50;
    if ( v39 >= 10 )
    {
        if ( v4 == 8 )
        {
            if ( !v3(17) || !v3(16) )
                goto LABEL_50;
            sub_406910(dword_C278E0, 100, 100);     // <-- 두 번째 이스터에그 표시
            while ( (sub_41F340() & 0x80000000) != 0 );
        }
        else
        {
            if ( v4 != 65 || !v3(83) || !v3(69) || !v3(32) || !v3(16) || v3(17) )
                goto LABEL_50;
            sub_406910(dword_C278E0, 100, 100);     // <-- 두 번째 이스터에그 표시
            while ( (sub_41F340() & 0x80000000) != 0 );
        }
        dword_A570EC = (int)sub_40E030;
        sub_41CE50();
        dword_45B36C = 1;
    }
    else
    {
        if ( v4 == 8 )
        {
            if ( !v3(17) || !v3(16) )
                goto LABEL_50;
            sub_406910(dword_C29B40, 100, 100);     // <-- 첫 번째 이스터에그 표시
            while ( (sub_41F340() & 0x80000000) != 0 );
        }
        else
        {
            if ( v4 != 65 || !v3(83) || !v3(69) || !v3(32) || !v3(16) || v3(17) )
                goto LABEL_50;
            sub_406910(dword_C29B40, 100, 100);     // <-- 첫 번째 이스터에그 표시
            while ( (sub_41F340() & 0x80000000) != 0 );
        }
        dword_A570EC = (int)sub_40E030;
        sub_41CE50();
    }
    LABEL_50:
    // ...
}
```

복잡해보이네요. 차근차근 분석해봅시다. 우선, 이스터에그 발동 조건이 충족되지 않으면 `goto LABEL_50;` 을 통해
키 조합을 검사하는 코드를 건너 뛰는 것으로 보입니다. `sub_406910(dword_C29B40, 100, 100);`가 이스터에그를 표시하는 함수로
보이네요. 어떤 조건일 때 호출되는지 확인해봅시다.

```c
if ( v39 >= 10 )
    // 두 번째 이스터에그
else
    if(v4 == 8)
        if(!v3(17) || !v3(16))
            goto LABEL_50;
            sub_406910(dword_C29B40, 100, 100);
    else
        if (v4 != 65 || !v3(83) || !v3(69) || !v3(32) || !v3(16) || v3(17))
            goto LABEL_50;
            sub_406910(dword_C29B40, 100, 100);
```

`v3`, `v4`가 무엇인지 알아야겠죠?

```c
v3 = GetAsyncKeyState;
```

`v3`은 GetAsyncKeyState를 가리킵니다.
[MSDN의 GetAsyncKeyState 문서](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getasynckeystate)에
따르면, 함수가 호출된 시점에 인자로 받은 가상 키 코드에 해당하는 키가 눌려있으면 적절한 값을 리턴한다고 합니다.
가상 키 코드에 대한 정보는 [MSDN의 Virtual-Key Codes 문서](https://learn.microsoft.com/en-us/windows/win32/inputdev/virtual-key-codes)를
참고하세요.

---

```c
v4 = sub_41F340();
```

`v4`는 `sub_41F340` 함수의 리턴값을 저장하네요. 자세히 분석해봅시다.

### sub_41F340

```c
WPARAM sub_41F340()
{
    // ...
    while ( 1 )
    {
        // ...
        if ( message == 256 )
        {
            v1 = sub_409550(Msg.wParam) == 0;
            result = Msg.wParam;
            if ( v1 )
            {
                switch ( Msg.wParam )
                {
                    case 8u:
                    case 9u:
                    case 0xDu:
                    case 0x15u:
                    case 0x1Bu:
                    case 0x21u:
                    case 0x22u:
                    case 0x25u:
                    case 0x26u:
                    case 0x27u:
                    case 0x28u:
                    case 0x2Eu:
                        return result;
                    default:
                        goto LABEL_14;
                }
            }
            return result;
        }
    LABEL_14:
        // ...
    }
    // ...
    return -2;
}
```

메시지 코드가 `WM_KEYDOWN (256)`일 때 wParam을 통해 어떤 키가 눌렸는지 알아냅니다.
키 코드가 `VK_BACK(8)`, `VK_TAB(9)`, `VK_ENTER(0xD)`, `VK_HANGUL(0x15)`, `VK_ESCAPE(0x1B)`, `VK_PRIOR(0x21)`,
`VK_NEXT(0x22)`, `VK_LEFT(0x25)`, `VK_UP(0x26)`, `VK_RIGHT(0x27)`, `VK_DOWN(0x28)`, `VK_DELETE(0x2E)` 중 하나라면
그 키 코드를 리턴합니다.

### sub_41F5D0 (2)

`v39`가 무엇인지 확인해봅시다.

```c
if ( v4 == 8 && v3(17) && v3(16) )
    ++v41;
if ( v41 <= 11 )
    goto LABEL_50;
if ( v4 == 73 )
    ++v40;
if ( v40 > 0 )
    if ( v4 == 78 )
        ++v38;
    if ( v38 > 1 && v4 == 79 )
        ++v39;
if ( v39 <= 0 )
        goto LABEL_50;
```

v3과 v4가 무엇인지 이미 분석을 해놨기 때문에 어떤 로직인지 간단하게 파악할 수 있을 것 같습니다. 위에서부터 하나씩
분석해보겠습니다.

1. `VK_CONTROL(17)`과 `VK_SHIFT(16)`이 눌려있는 상태로 `VK_BACK(8)`을 누르면 v41 값이 1만큼 증가합니다.

2. v41이 11 이하라면 `goto LABEL_50;`을 통해 키 조합을 확인하는 로직을 벗어납니다.
즉, Control키와 Shift키를 누른 상태에서 Backspace를 12번 이상 눌러야 나머지 코드가 실행됩니다.

3. `I(73)`를 누르면 v40 값이 1 증가합니다.

4. v40가 0을 초과하는 상태에서, 다시 말해 I를 1회 이상 누른 상태에서 `N(78)`을 누르면 v38의 값이 1만큼 증가합니다.

5. v38이 1을 초과하는 상태에서, 다시 말해 N을 2회 이상 누른 상태에서 `O(79)`를 누르면 v39의 값이 1 증가합니다.

여기까지가 두 이스터에그 모두에 해당하는 키 조합입니다. 정리하자면 Ctrl + Shift를 누른 상태에서 Backspace를 12번 이상
누르고, I 1번 이상, N 2번 이상, O 1번 이상 누르면 됩니다. 그런데 INNO가 무슨 의미일까요? 이스터에그에 나오는 개발자의
이메일 주소가 `innoboy@nownuri.net`인 것으로 보아 개발자가 사용하는 id의 앞부분인 것 같습니다.

아무튼, 첫 번째 이스터에그를 표시하는 로직을 다시 확인해봅시다.

```c
if ( v39 >= 10 )
    // 두 번째 이스터에그
else
    if(v4 == 8)
        if(!v3(17) || !v3(16))
            goto LABEL_50;
        sub_406910(dword_C29B40, 100, 100);
    else
        if (v4 != 65 || !v3(83) || !v3(69) || !v3(32) || !v3(16) || v3(17))
            goto LABEL_50;
        sub_406910(dword_C29B40, 100, 100);
```

이스터에그가 발동되기 위해서는 v39가 10 미만이어야 하며, `v4 == 8`이고 `!v3(17) || !v3(16)`이어야 합니다.
쉽게 설명하자면, O가 10번 미만 눌리고 Control과 Shift가 눌려있는 상태에서 Backspace 키를 눌러야 이스터에그가 발동됩니다.

`v4 == 8`가 아닐 때 발동되는 코드도 봅시다. if문 안의 조건이 참일 때 `goto LABEL_50;`을 해버리므로 조건에 NOT을 씌워야겠죠?
`v4 == 65 && v3(83) && v3(69) && v3(32) && v3(16) && !v3(17)` 일 때 이스터에그가 발생됩니다.
다시 말해, S, E, SPACE, SHIFT가 눌렸고 Control이 눌리지 않은 상태에서 A를 누르면 이스터에그가 발생됩니다. 이 로직은 이번에
리버싱을 다시 하면서 알게 됐네요. 전에는 왜 발견하지 못했을까요...

두 번째 이스터에그를 표시하는 로직을 봅시다.

```c
if ( v39 >= 10 )
    if ( v4 == 8 )
        if ( !v3(17) || !v3(16) )
            goto LABEL_50;
        sub_406910(dword_C278E0, 100, 100);
    else
        if ( v4 != 65 || !v3(83) || !v3(69) || !v3(32) || !v3(16) || v3(17) )
            goto LABEL_50;
        sub_406910(dword_C278E0, 100, 100);
```

나머지는 다 똑같은데 v39가 10 이상이어야 합니다. 즉, O를 10번 이상 눌러야 됩니다.

### 결론
이스터에그를 띄우기 위한 키 조합은 다음과 같습니다.

1. Control + Shift를 누른 상태에서 Backspace를 12번 이상 누릅니다.
2. I를 한 번 이상 누릅니다.
3. N을 두 번 이상 누릅니다.
4. O를 누릅니다. 10번 미만 누르면 첫 번째 이스터에그가, 10번 이상 누르면 두 번째 이스터에그가 표시됩니다.
5. Ctrl + Shift를 누른 상태에서 Backspace를 누르거나, S + E + Shift + Space를 누르고 Control을 누르지 않은 상태에서 A를 누릅니다.

## 리버싱을 마치며
re3이라고 GTA3를 리버싱 한 다음 분석한 내용을 토대로 소스코드를 새롭게 작성하여 만들어진 오픈소스(?) GTA3가 있는데요,
한컴타자연습의 바이너리가 단순하다보니 보름정도 시간을 투자하면 오픈소스 한컴타자연습을 만들 수 있을 것 같습니다.
하지만 저작권 문제가 있으니 이스터에그를 찾는것으로 만족하겠습니다.

이 글에는 이스터에그에 관한 분석 내용만 적었는데요, 사실 세이브파일의 구조나 전반적인 프로그램의 구성도 대충 분석해봤습니다.
제가 WinAPI 개발을 해본 적이 없어서 코드를 평가할 자격이 없는데요, 코드가 꽤나 창의적인(?) 방법으로 작성됐다고 생각합니다.
예를 들자면, 화면에 버튼을 추가할 때 일반적으로 CreateWindow 함수를 사용한다고 알고 있습니다. 그런데 한컴타자연습은
화면에 버튼을 이미지로 그린 다음, 마우스 클릭 이벤트가 발생하면 어느 좌표에서 클릭했는지를 토대로 어떤 버튼이 클릭됐는지
알아내고, 그에 맞는 동작을 하도록 프로그래밍 되어 있습니다. 아무튼 잘 작동하기만 하면 되는거겠죠?

사실 제가 글 쓰는 것을 잘 못합니다. 머릿속에 하고 싶은 말은 많은데, 그걸 글로 옮기면 이해하기 힘든 괴상한 문장이 나오고
한번 쓰기 시작한 단어나 표현을 계속 반복해서 씁니다. 이 글이 블로그에 올라가는 제대로 된 첫 글일텐데, 사실 쓰다가 도저히
안되겠어서 갖다 버린 글이 두 개나 되고, 이 글도 표현을 다듬느라 일주일정도 붙들었습니다. 잘 이해 안 되는 부분이나 궁금한
점은 댓글을 남겨주시면 감사하겠습니다.