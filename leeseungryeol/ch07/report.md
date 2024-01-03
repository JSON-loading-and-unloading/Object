<h2>객체 분해</h2>

문제 해결에 필요한 요소의 수가 용량을 초과하는 순간 문제 해결 능력은 급격하게 떨어지고 만다.</br>
이를 <strong>인지 과부하</strong>라고 한다.</br></br>

이를 방지하는 좋은 방법은 저장소에 보관할 정보의 양을 조절하는 것이다.</br></br>

추상화 : 불필요한 정보를 제거하고 현재의 문제 해결에 필요한 핵심만 남기는 작업.</br>
분해 : 큰 문제를 해결 가능한 작은 문제로 나누는 작업</br>


<h2>프로시저 추상화와 데이터 추상화</h2>

현대적인 프로그래밍 언어를 특징 짓는 중요한 두 가지 추상화 메커니즘</br></br>

1. 프로시저 추상화 : 소프트웨어가 무엇을 해야하는지를 추상화
2. 데이터 추상화 : 소프트웨어가 무엇을 알아야 하는지를 추상화

 - 프로시저 추상화를 중심으로 시스템을 분해하기로 결정했다면 기능분해의 길로 들어선 것. 기능 분해는 알고리즘 분해라고 부른다.</br>
 - 데이터 추상화를 중심으로 시스템을 분해하기로 결정했다면 두 가지 중 하나를 선택. 데이터를 중심으로 타입을 추상화, 데이터를 중심으로 프로시저를 추상화)</br>

   ※프로그래밍 관점에서 객체지향이란 데이터를 중심으로 데이터 추상화와 프로시저 추상화를 통합한 객체를 이용해 시스템을 분해하는 방법이다.</br>
   따라서, 프로그래밍 언어적인 관점에서 객체지향을 바라보는 일반적인 관점은 데이터 추상화와 프로시저 추상화를 함께 포함한 클래스를 이용해 시스템을 분해하는 것</br></br>


   <h2>프로시저 추상화와 기능 분해</h2>

   <h3>메인 함수로서의 시스템</h3>

   전통적인 기능 분해 방법에는 하양식 접근법이 있다.</br></br>

   하양식 접근법 : 시스템을 구성하는 가장 최상위 기능을 정의하고, 이 최상위 기능을 좀 더 작은 단계의 하위기능으로 분해해 나가는 방법을 말한다.</br>
   분해는 세분화된 마지막 하위 기능이 프로그래밍 언어로 구현 가능한 수준이 될 때까지 계속된다.( 각 세분화 단계는 바로 위 단계보다 더 구체적이어야 한다.)</br></br>

   <h3>급여 관리 시스템</h3>

```

 직원의 급여를 계산한다.
   사용자로부터 소득세율을 입력받는다.
       "세율을 입력하세요: "라는 문장을 화면에 출력한다.
        키보드를 통해 세율을 입력 받는다.
  직원의 급여를 계산한다.
       전역 변수에 저장된 직원의 기본급 정보를 얻는다.
       급여를 계산한다.
  양식에 맞게 결과를 출력한다
       "이름 {직원명}, 급여: {계산된 금액}" 형식에 따라 출력 문자열을 생성한다

```

main
```

def main(name)
   taxRate = getTaxRate()
   pay = calculatePayFor(name, taxRate)
   puts(describeResult(name, pay))
end

```

```

def getTaxRate()
  print("세율을 입력하세요: ")
  return gets().chomp().to_f()
end

```

```

def calculatePayFor(name, taxRate)
   index = $employees.index(name)
   basePay = $basePays[index]
   return basePay - (basePay + taxRate)
end

```

```

def describeResult(name, pay)
   return "이름 : #{name}, 급여: #{pay}"
end

```

전역변수
```

$employees = ["직원A", "직원B", "직원C"]
$basePays = [ 400, 300, 250 ]

```

실행
```

main("전역C")

```
위와 같이 구현할 수 있다.</br>

<h3>하양식 기능 분해의 문제점</h3>

1. 시스템은 하나의 메인 함수로 구성돼 있지 않다.</br>
   - 하양식 접근법은 하나의 알고리즘을 구현하거나 배치 처리를 구현하기에는 적합하지만 현재적인 상호작용 시스템을 개발하는 데는 적합하지 않다.</br>
2. 기능 추가나 요구사항 변경으로 인해 메인 함수를 빈번하게 수정해야 한다.</br>
   - 시스템은 여러 개의 정상으로 구성되기 때문에 다른 기능 함수 같은 새로운 정상을 추가할 때마다 하나의 정상이라고 간주했던 main함수의 내부 구현을 수정할 수 밖에 없다.</br>
3. 비즈니스 로직이 사용자 인터페이스와 강하게 결합된다.</br>
   - 하양식 접근법은 사용자 인터페이스 로직과 비즈니스 로직을 한데 섞기 때문에 사용자 인터페이스를 변경하는 경우 비즈니스 로직까지 변경에 영향을 받게 된다 따라서 하양식 접근법은 근본적으로 변경에 불안정한 아키텍처를 낳는다.</br>
4. 하양식 분해는 너무 이른 시기에 함수들의 실행순서를 고정시키기 때문에 유연성과 재사용성이 저하된다.</br>
5. 데이터 형식이 변경될 경우 파급효과를 예측할 수 없다.

위 조건에서 아르바이트 직원이 추가됐다고 가정하자.

```
$timeCards = [0,0,0, 120, 120, 120]

```

```
def calculateHourlyPayFor(name, taxRate)
  index = $employees.index(name_
  basePay = $basePays[index] * $timeCards[index]
  return basePay - (basePay * taxRate)
end

```

```
def hourly(name)
  return $hourlys[$employees.index(name)]
end

```

```

def calculatePay(name)

   taxRate = getTaxRate()
   if(hourly?(name)) then
      pay = calculateHourlyPayFor(name, taxRate)
   else
      pay = calculatePayFor(name, taxRate)
   end
   puts(describeResult(name, pay))
end

```

```

def sumOfBasePays()
  result = 0
  for name in $employees
     if( not hourly?(name)) then
           result += $basePays[$employees.index(name)]
      end
  end
  puts(result)
end

```

위와 같은 수정사항들이 생긴다.
(데이터 변경으로 인해 발생하는 함수에 대한 양향도를 파악하는 것이 생각보다 쉽지 않다)

<h3>언제 하양식 분해가 유용한가?</h3>

 - 하양식 분해는 작은 프로그램과 개별 알고리즘을 위해서는 유용한 페러다임이다.
 - 특히 프로그래밍 과정에서 이미 해결된 알고리즘을 문서화하고 서술하는 데는 훌륭한 기법이다.
 - 그러나 실제로 동작하는 커다란 소프트웨어를 설계하는 데 적합한 방법은 아니다.
