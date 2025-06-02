## **1️⃣** 

## **ScrollView + VStack + ForEach**

## **로 구조 변경!**

  

### **왜?**

- **List**는 시스템 기본 스타일(특히 배경/간격/둥글기)이 강제됨
    
- **ScrollView + VStack**을 쓰면,
    
    각 카드(View)에 원하는 대로
    
    .background(Color...), .cornerRadius(20), .shadow, .padding() 등을
    
    **완벽하게 커스텀**할 수 있습니다!

아래 리스트를
```
List {
    ForEach(filteredNotes, id: \.id) { ... }
}
```

아래와 같이 변경
```
ScrollView {
    VStack(spacing: 12) {
        ForEach(filteredNotes, id: \.id) { ... }
    }
    .padding(.vertical)
}
```

그리고 - ForEach 안의 **LearningNoteSubView**를
    
    카드처럼 감싸서- ForEach 안의 **LearningNoteSubView**를
    
    카드처럼 감싸서

```
LearningNoteSubView(...)
    .padding()
    .background(Color(.systemPink).opacity(0.15))
    .cornerRadius(20)
    .shadow(color: .black.opacity(0.06), radius: 6, y: 2)
```

## **3️⃣** 

## **추가 팁**

- 카드 사이 간격은 **VStack의 spacing**으로 조절
    
- 카드 바깥쪽 간격은 **.padding(.horizontal)**로 조절
    

---

## **4️⃣** 

## **정리**

- **List**는 기본 스타일이 강하고, 커스텀 한계가 있다
    
- **ScrollView + VStack**은 카드 디자인에 최적,
    
    실무에서도 거의 항상 이 방식 사용!
    

---

**이 구조로 적용하시면**

**딱 원하시는 둥글둥글 카드 UI가 아주 쉽게 구현됩니다!**