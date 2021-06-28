# Generic
- 클래스 내부에서 사용할 데이터 타입을 밖에서 저장하는 기법

## 장점
- 사용할 데이터 타입(refernce-type)을 외부에서 정의할 수 있다.
- 사용할 데이터 타입을 제한하기 위해서는 선언 부에 예를 들어 `<T extends Number>` 와 같이 제한하면 된다. (잘못된 데이터 타입을 정의하면 컴파이 에러 발생)

```java
public class GenericTest{
    public static void main(String[] args) {
        Generic_sub<Integer> gs = new Generic_sub<>();
        Generic_sub<String> gs1 = new Generic_sub<>();

        gs.setParam1(10);
        gs.setParam2(20);

        gs1.setParam1("홍길동");
        gs1.setParam2("김철수");

        System.out.println(gs.printParamNum(gs.getParam1(),gs.getParam2()));
        System.out.println(gs.printParamNum(gs1.getParam1(),gs1.getParam2()));
    }
}

class Generic_sub<T>{
    private T param1;
    private T param2;

    public T getParam1() {
        return param1;
    }

    public T getParam2() {
        return param2;
    }

    public void setParam1(T param1) {
        this.param1 = param1;
    }

    public void setParam2(T param2) {
        this.param2 = param2;
    }

    public <T> StringBuffer printParamNum(T param1, T param2){
        StringBuffer sb = new StringBuffer();
        sb.append(param1);
        sb.append(param2);
        return sb;
    }
}
```
