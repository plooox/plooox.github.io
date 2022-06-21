# [Clean Code] Chap10. 클래스


<!--more-->

# 클래스

## 클래스 체계

- 표준 자바 컨벤션
  1. 변수 목록: `static public` → `static private` → `private 인스턴스` → `public(거의 안씀)`
  2. 공개함수
  3. 비공개함수: 자신을 호출하는 공개함수 직후

⇒ **추상화 단계가 순차적으로 내려간다**

### 캡슐화

- 변수와 유틸리티 함수 ← 공개하지 않는게 좋으나 ‘반드시' 숨길 필요는 없다.
  - `protected` 로 선언 후 테스트 코드에 접근 허용
- 그럼에도 불구하고 비공개 상태를 유지하기 위해 온갖 방법을 강구하는 것이 최우선
  - 캡슐화 해제는 언제나 최후 수단

## 클래스는 작아야 한다!

- 함수와 마찬갖지로 ‘작게'가 기본 규칙

  - ‘얼마나' 작아야 할까? ⇒ **맡은 책임을 측정**
    <details>
    <summary>너무 많은 책임</summary>
    <div markdown="1">

    ```java
    public class SuperDashboard extends JFrame implements MetaDataUser {
    	public String getCustomizerLanguagePath()
    	public void setSystemConfigPath(String systemConfigPath)
    	public String getSystemConfigDocument()
    	public void setSystemConfigDocument(String systemConfigDocument)
    	public boolean getGuruState()
    	public boolean getNoviceState()
    	public boolean getOpenSourceState()
    	public void showObject(MetaObject object)
    	public void showProgress(String s)
    	public boolean isMetadataDirty()
    	public void setIsMetadataDirty(boolean isMetadataDirty)
    	public Component getLastFocusedComponent()
    	public void setLastFocused(Component lastFocused)
    	public void setMouseSelectState(boolean isMouseSelected)
    	public boolean isMouseSelected()
    	public LanguageManager getLanguageManager()
    	public Project getProject()
    	public Project getFirstProject()
    	public Project getLastProject()
    	public String getNewProjectName()
    	public void setComponentSizes(Dimension dim)
    	public String getCurrentDir()
    	public void setCurrentDir(String newDir)
    	public void updateStatus(int dotPos, int markPos)
    	public Class[] getDataBaseClasses()
    	public MetadataFeeder getMetadataFeeder()
    	public void addProject(Project project)
    	public boolean setCurrentProject(Project project)
    	public boolean removeProject(Project project)
    	public MetaProjectHeader getProgramMetadata()
    	public void resetDashboard()
    	public Project loadProject(String fileName, String projectName)
    	public void setCanSaveMetadata(boolean canSave)
    	public MetaObject getSelectedObject()
    	public void deselectObjects()
    	public void setProject(Project project)
    	public void editorAction(String actionName, ActionEvent event)
    	public void setMode(int mode)
    	public FileManager getFileManager()
    	public void setFileManager(FileManager fileManager)
    	public ConfigManager getConfigManager()
    	public void setConfigManager(ConfigManager configManager)
    	public ClassLoader getClassLoader()
    	public void setClassLoader(ClassLoader classLoader)
    	public Properties getProps()
    	public String getUserHome()
    	public String getBaseDir()
    	public int getMajorVersionNumber()
    	public int getMinorVersionNumber()
    	public int getBuildNumber()
    	public MetaObject pasting(MetaObject target, MetaObject pasted, MetaProject project)
    	public void processMenuItems(MetaObject metaObject)
    	public void processMenuSeparators(MetaObject metaObject)
    	public void processTabPages(MetaObject metaObject)
    	public void processPlacement(MetaObject object)
    	public void processCreateLayout(MetaObject object)
    	public void updateDisplayLayer(MetaObject object, int layerIndex)
    	public void propertyEditedRepaint(MetaObject object)
    	public void processDeleteObject(MetaObject object)
    	public boolean getAttachedToDesigner()
    	public void processProjectChangedState(boolean hasProjectChanged)
    	public void processObjectNameChanged(MetaObject object)
    	public void runProject()
    	public void setAçowDragging(boolean allowDragging)
    	public boolean allowDragging()
    	public boolean isCustomizing()
    	public void setTitle(String title)
    	public IdeMenuBar getIdeMenuBar()
    	public void showHelper(MetaObject metaObject, String propertyName)

    	// ... many non-public methods follow ...
    }
    ```

    </div>
    </details>

    <details>
    <summary>메서드 수를 줄인다면?</summary>
    <div markdown="1">

    ```java
    public class SuperDashboard extends JFrame implements MetaDataUser {
    	// 책임1. Swing 컴포넌트 관리
    	public Component getLastFocusedComponent()
    	public void setLastFocused(Component lastFocused)

    	// 책임2. 소프트웨어 버전 추적
    	public int getMajorVersionNumber()
    	public int getMinorVersionNumber()
    	public int getBuildNumber()
    }
    ```

    - 메서드 수가 작음에도 불구하고 **책임이 너무 많다!**
      - 버전 관리를 담당하는 메서드를 따로 분리하여 `Version` 클래스를 만들 수 있다
        ```java
        public class Version {
        	public int getMajorVersionNumber()
        	public int getMinorVersionNumber()
        	public int getBuildNumber()
        }
        ```

    </div>
    </details>

  - 클래스 이름은 해당 클래스의 책임을 기술해야한다.
    - 클래스 이름이 모호하거나 짓기 어렵다면, 책임이 너무 큰 것
    - 25단어 내외로 클래스 설명이 가능해야 함

### 단일 책임 원칙(SRP)

- 클래스나 모듈을 **변경할 이유(책임)**가 단 하나뿐이어야 한다.
- 객체 지향 설계의 핵심
- 이해하고 지키기 쉬우나 가장 무시하는 규칙
  → why? ) ‘돌아가는 소프트웨어' 개발에 초점이 맞춰지기 때문
- 작은 클래스 여럿으로 이뤙진 시스템이 바람직하다

### 응집도

- **클래스는 인스턴스 변수 수가 작아야 한다.**
- 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 한다.
  - 메서드가 변수를 더 많이 사용할 수록 메서드와 클래스는 응집도가 더 높다
- 가능한한 응집도가 높은 클래스를 지향
  - **응집도가 높다 = 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다**

```java
// 응집도가 높은 클래스
public class Stack {
	private int topOfStack = 0;
	List<Integer> elements = new LinkedList<Integer>();

	public int size() {
		return topOfStack;
	}

	public void push(int element) {
		topOfStack++;
		elements.add(element);
	}

	public int pop() throws PoppedWhenEmpty {
		if (topOfStack == 0)
			throw new PoppedWhenEmpty();
		int element = elements.get(--topOfStack);
		elements.remove(topOfStack);
		return element;
	}
}
```

- **함수는 작게, 매개변수 목록은 짧게**
  - 메서드가 사용하는 인스턴스 변수가 많아진다 → 새로운 클래스로 쪼개라!

### 응집도를 유지하면 작은 클래스 여럿이 나온다

Ex) 변수가 아주 많은 큰 함수

  <details>
  <summary>예시</summary>
  <div markdown="1">

    ```java
    package literatePrimes;

    public class PrintPrimes {
      public static void main(String[] args) {
        final int M = 1000;
        final int RR = 50;
        final int CC = 4;
        final int WW = 10;
        final int ORDMAX = 30;
        int P[] = new int[M + 1];
        int PAGENUMBER;
        int PAGEOFFSET;
        int ROWOFFSET;
        int C;
        int J;
        int K;
        boolean JPRIME;
        int ORD;
        int SQUARE;
        int N;
        int MULT[] = new int[ORDMAX + 1];

        J = 1;
        K = 1;
        P[1] = 2;
        ORD = 2;
        SQUARE = 9;

        while (K < M) {
          do {
            J = J + 2;
            if (J == SQUARE) {
              ORD = ORD + 1;
              SQUARE = P[ORD] * P[ORD];
              MULT[ORD - 1] = J;
            }
            N = 2;
            JPRIME = true;
            while (N < ORD && JPRIME) {
              while (MULT[N] < J)
                MULT[N] = MULT[N] + P[N] + P[N];
              if (MULT[N] == J)
                JPRIME = false;
              N = N + 1;
            }
          } while (!JPRIME);
          K = K + 1;
          P[K] = J;
        }
        {
          PAGENUMBER = 1;
          PAGEOFFSET = 1;
          while (PAGEOFFSET <= M) {
            System.out.println("The First " + M + " Prime Numbers --- Page " + PAGENUMBER);
            System.out.println("");
            for (ROWOFFSET = PAGEOFFSET; ROWOFFSET < PAGEOFFSET + RR; ROWOFFSET++) {
              for (C = 0; C < CC;C++)
                if (ROWOFFSET + C * RR <= M)
                  System.out.format("%10d", P[ROWOFFSET + C * RR]);
              System.out.println("");
            }
            System.out.println("\f"); PAGENUMBER = PAGENUMBER + 1; PAGEOFFSET = PAGEOFFSET + RR * CC;
          }
        }
      }
    }
    ```

  </div>
  </details>

→ 큰 함수 일부를 작은 함수로 분할

→ 추출하고자 하는 코드가 큰 함수에 정의된 변수를 많이 사용

→ 변수를 클래스 인스턴스 변수로 승격시켜 함수 인수가 필요없게 구성 (응집력 down)

→ 몇몇 함수가 몇몇 인스턴스 변수만 사용하면 독자적인 클래스로 분리

  <details>
  <summary>리펙토링</summary>
  <div markdown="1">

```java
package literatePrimes;

// PrimePrinter 클래스: main함수 하나만 포함, 실행 환경을 책임
public class PrimePrinter {
	public static void main(String[] args) {
		final int NUMBER_OF_PRIMES = 1000;
		int[] primes = PrimeGenerator.generate(NUMBER_OF_PRIMES);

		final int ROWS_PER_PAGE = 50;
		final int COLUMNS_PER_PAGE = 4;
		RowColumnPagePrinter tablePrinter =
			new RowColumnPagePrinter(ROWS_PER_PAGE,
						COLUMNS_PER_PAGE,
						"The First " + NUMBER_OF_PRIMES + " Prime Numbers");
		tablePrinter.print(primes);
	}
}
```

```java
package literatePrimes;

import java.io.PrintStream;

// RowColumnPagePrinter 클래스: 숫자목록을 행과 열에 맞추어 페이지를 출력
public class RowColumnPagePrinter {
	private int rowsPerPage;
	private int columnsPerPage;
	private int numbersPerPage;
	private String pageHeader;
	private PrintStream printStream;

	public RowColumnPagePrinter(int rowsPerPage, int columnsPerPage, String pageHeader) {
		this.rowsPerPage = rowsPerPage;
		this.columnsPerPage = columnsPerPage;
		this.pageHeader = pageHeader;
		numbersPerPage = rowsPerPage * columnsPerPage;
		printStream = System.out;
	}

	public void print(int data[]) {
		int pageNumber = 1;
		for (int firstIndexOnPage = 0 ;
			firstIndexOnPage < data.length ;
			firstIndexOnPage += numbersPerPage) {
			int lastIndexOnPage =  Math.min(firstIndexOnPage + numbersPerPage - 1, data.length - 1);
			printPageHeader(pageHeader, pageNumber);
			printPage(firstIndexOnPage, lastIndexOnPage, data);
			printStream.println("\f");
			pageNumber++;
		}
	}

	private void printPage(int firstIndexOnPage, int lastIndexOnPage, int[] data) {
		int firstIndexOfLastRowOnPage =
		firstIndexOnPage + rowsPerPage - 1;
		for (int firstIndexInRow = firstIndexOnPage ;
			firstIndexInRow <= firstIndexOfLastRowOnPage ;
			firstIndexInRow++) {
			printRow(firstIndexInRow, lastIndexOnPage, data);
			printStream.println("");
		}
	}

	private void printRow(int firstIndexInRow, int lastIndexOnPage, int[] data) {
		for (int column = 0; column < columnsPerPage; column++) {
			int index = firstIndexInRow + column * rowsPerPage;
			if (index <= lastIndexOnPage)
				printStream.format("%10d", data[index]);
		}
	}

	private void printPageHeader(String pageHeader, int pageNumber) {
		printStream.println(pageHeader + " --- Page " + pageNumber);
		printStream.println("");
	}

	public void setOutput(PrintStream printStream) {
		this.printStream = printStream;
	}
}
```

```java
package literatePrimes;

import java.util.ArrayList;

// PrimeGenerator 클래스: 소수 목록을 생성
public class PrimeGenerator {
	private static int[] primes;
	private static ArrayList<Integer> multiplesOfPrimeFactors;

	protected static int[] generate(int n) {
		primes = new int[n];
		multiplesOfPrimeFactors = new ArrayList<Integer>();
		set2AsFirstPrime();
		checkOddNumbersForSubsequentPrimes();
		return primes;
	}

	private static void set2AsFirstPrime() {
		primes[0] = 2;
		multiplesOfPrimeFactors.add(2);
	}

	private static void checkOddNumbersForSubsequentPrimes() {
		int primeIndex = 1;
		for (int candidate = 3 ; primeIndex < primes.length ; candidate += 2) {
			if (isPrime(candidate))
				primes[primeIndex++] = candidate;
		}
	}

	private static boolean isPrime(int candidate) {
		if (isLeastRelevantMultipleOfNextLargerPrimeFactor(candidate)) {
			multiplesOfPrimeFactors.add(candidate);
			return false;
		}
		return isNotMultipleOfAnyPreviousPrimeFactor(candidate);
	}

	private static boolean isLeastRelevantMultipleOfNextLargerPrimeFactor(int candidate) {
		int nextLargerPrimeFactor = primes[multiplesOfPrimeFactors.size()];
		int leastRelevantMultiple = nextLargerPrimeFactor * nextLargerPrimeFactor;
		return candidate == leastRelevantMultiple;
	}

	private static boolean isNotMultipleOfAnyPreviousPrimeFactor(int candidate) {
		for (int n = 1; n < multiplesOfPrimeFactors.size(); n++) {
			if (isMultipleOfNthPrimeFactor(candidate, n))
				return false;
		}
		return true;
	}

	private static boolean isMultipleOfNthPrimeFactor(int candidate, int n) {
		return candidate == smallestOddNthMultipleNotLessThanCandidate(candidate, n);
	}

	private static int smallestOddNthMultipleNotLessThanCandidate(int candidate, int n) {
		int multiple = multiplesOfPrimeFactors.get(n);
		while (multiple < candidate)
			multiple += 2 * primes[n];
		multiplesOfPrimeFactors.set(n, multiple);
		return multiple;
	}
}
```

- 프로그램 길이가 길어짐
  - 서술적인 변수 이름 사용
  - 코드에 주석을 추가하는 수단 → 함수 선언 & 클래스 선언
  - 가독성을 위한 공백 추가
- 기존 프로그램의 로직을 그대로 두고 리펙토링 수행

  - 프로그램 동작을 검증하는 테스트 슈트 작성
  - 한번에 하나씩 수차례에 거쳐 코드변경
    - 코드를 변경할 때마다 테스트를 수행

  </div>
  </details>

## 변경하기 쉬운 클래스

- 대다수 시스템은 지속적인 변경이 가해짐
- 무언가를 변경할 때마다 시스템이 의도대로 동작하지 않을 위험 존재
- 깨끗한 시스템 → 클래스를 체계적으로 정리, 변경에 수반되는 위험을 낮춤
  <details>
  <summary>변경이 필요한 코드</summary>
  <div markdown="1">

  ```java
  public class Sql {
  	public Sql(String table, Column[] columns)
  	public String create()
  	public String insert(Object[] fields)
  	public String selectAll()
  	public String findByKey(String keyColumn, String keyValue)
  	public String select(Column column, String pattern)
  	public String select(Criteria criteria)
  	public String preparedInsert()
  	private String columnList(Column[] columns)
  	private String valuesList(Object[] fields, final Column[] columns) private String selectWithCriteria(String criteria)
  	private String placeholderList(Column[] columns)
  }
  ```

  - 새로운 SQL문을 지원하거나 SQL문을 수정할 때 수정이 필요함 → **SRP 위반**

</div>
</details>

- 클래스 일부에서만 사용되는 비공개 메서드는 코드 개선의 잠재적인 여지를 시사
  <details>
  <summary>닫힌 클래스 집합</summary>
  <div markdown="1">

  ```java
  	abstract public class Sql {
  		public Sql(String table, Column[] columns)
  		abstract public String generate();
  	}
  	public class CreateSql extends Sql {
  		public CreateSql(String table, Column[] columns)
  		@Override public String generate()
  	}

  	public class SelectSql extends Sql {
  		public SelectSql(String table, Column[] columns)
  		@Override public String generate()
  	}

  	public class InsertSql extends Sql {
  		public InsertSql(String table, Column[] columns, Object[] fields)
  		@Override public String generate()
  		private String valuesList(Object[] fields, final Column[] columns)
  	}

  	public class SelectWithCriteriaSql extends Sql {
  		public SelectWithCriteriaSql(
  		String table, Column[] columns, Criteria criteria)
  		@Override public String generate()
  	}

  	public class SelectWithMatchSql extends Sql {
  		public SelectWithMatchSql(String table, Column[] columns, Column column, String pattern)
  		@Override public String generate()
  	}

  	public class FindByKeySql extends Sql public FindByKeySql(
  		String table, Column[] columns, String keyColumn, String keyValue)
  		@Override public String generate()
  	}

  	public class PreparedInsertSql extends Sql {
  		public PreparedInsertSql(String table, Column[] columns)
  		@Override public String generate() {
  		private String placeholderList(Column[] columns)
  	}

  	public class Where {
  		public Where(String criteria) public String generate()
  	}

  	public class ColumnList {
  		public ColumnList(Column[] columns) public String generate()
  	}
  ```

  - 공개 인터페이스 → 파생 클래스
  - 비공개 메서드 → 파생 클래스로 이동
  - 모든 파생 클래스가 사용하는 비공개 메서드 → 유틸리티 클래스(`Where`, `ColumnList`)

  - 각 클래스가 단순하고 이해하기 쉬움
  - 함수 하나를 수정해도 다른 함수가 고장날 위험도 배제
  - 테스트 관점에서 모든 논리 증명에도 용이
  - OCP원칙 준수

</div>
</details>

### 변경으로부터 격리

- concrete 클래스: 구체적인 클래스, 상세한 구현을 포함
- abstract 클래스: 추상 클래스, 개념만 포함
- **인터페이스**와 **추상클래스**를 사용해 구현이 미치는 영향을 격리

  - 상셓나 구현에 의존하는 코드는 테스트가 어려움
  - 추상화를 통해 테스트가 가능할 정도로 시스템의 결합도를 낮춤으로서 유연성과 재사용성을 높임

- <details>
  <summary>예시</summary>
  <div markdown="1">

  ```java
  // 인터페이스(StockExchange)를 생성한 후 메서드를 선언
  public interface StockExchange {
    Money currentPrice(String symbol);
  }
  ```

  ```java
  // StockExchange 인터페이스를 구현하는 TokyoStockExchange 클래스를 구현
  public Portfolio {
    private StockExchange exchange;
    public Portfolio(StockExchange exchange) {
      this.exchange = exchange;
    }
    // ...
  }
  ```

  ```java
  // 테스트용 클래스(FixedStockExchangeStub) By TokyoStockExchange 클래스
  public class PortfolioTest {
    private FixedStockExchangeStub exchange;
    private Portfolio portfolio;

    @Before
    protected void setUp() throws Exception {
      exchange = new FixedStockExchangeStub();
      exchange.fix("MSFT", 100);
      portfolio = new Portfolio(exchange);
    }

    @Test
    public void GivenFiveMSFTTotalShouldBe500() throws Exception {
      portfolio.add(5, "MSFT");
      Assert.assertEquals(500, portfolio.value());
    }
  }
  ```

  </div>
  </details>

