# Eel（一个用于将 Python 和 JavaScript（通过网页）结合的库）
eel.spawn 是 Eel 提供的一种让 Python 函数在 JavaScript 中异步执行的方法。它允许您在不阻塞 JavaScript 执行的情况下调用一个 Python 函数，并获得异步的非阻塞执行效果。这对于处理耗时操作而不想阻塞前端界面很有帮助。
# 使用 eel.spawn 的场景和方法：
1. 异步操作：
    * eel.spawn 提供了一种方式来在不等待函数返回结果的情况下调用 Python 函数。这样可以让 JavaScript 继续执行其他任务，同时后台运行 Python 代码。
2. 避免阻塞：
    * 在需要长时间运行的 Python 代码或需要后台任务时，使用 eel.spawn 是理想的选择，因为它不会阻塞 JavaScript 的事件循环。
3. 基本用法： eel.spawn(python_function, arg1, arg2, ...);
    * python_function 是您希望调用的 Python 函数。
    * arg1, arg2, … 是传递给该 Python 函数的参数。
4. 区别于其他调用方法：
    * 与同步调用不同，使用 eel.spawn 不会立即返回结果，也不使用 await。
    * 不会直接从 spawn 调用中获取返回值。如果需要处理返回值，通常需要从 Python 端通过回调或者其他通信机制（如 Eel 的回调功能）与 JavaScript 进行通信。
# Eel中的JavaScript
在 JavaScript 中，Promise 是一种用于处理异步操作的对象。它代表一个尚未完成但最终会成功或失败的操作，并相应地返回一个结果。Promise 提供了一种更加结构化的方式来处理异步代码，而不是使用回调函数，这可以帮助避免“回调地狱”。
Promise 有以下几个关键特点：
1. 状态：
    * Pending（待定）：初始状态，表示异步操作尚未完成。
    * Fulfilled（已完成）：表示异步操作成功完成，结果可以通过 .then() 方法获得。
    * Rejected（已拒绝）：表示异步操作失败，错误信息可以通过 .catch() 方法获得。
2. 方法：
    * .then(onFulfilled, onRejected): 用于添加在 Promise 完成时要执行的回调函数。
    * .catch(onRejected): 用于添加在 Promise 被拒绝时要执行的回调函数。
    * .finally(onFinally): 无论 Promise 的结果如何（完成或拒绝），都会执行的回调函数。
3. 链式调用：因为 then() 和 catch() 都返回一个新的 Promise，所以你可以使用链式调用来组织代码，使其更加清晰和易于管理。
下面是一个使用 Promise 的简单示例：
function asyncOperation() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const success = true;  // 假设操作成功
      if (success) {
        resolve("操作成功");
      } else {
        reject("操作失败");
      }
    }, 1000);
  });
}

asyncOperation()
  .then(result => {
    console.log(result);  // 输出: 操作成功
  })
  .catch(error => {
    console.error(error);
  })
  .finally(() => {
    console.log("操作完成，无论成功或失败");
  });

在这个示例中，asyncOperation 函数返回一个 Promise，模拟一个耗时的异步操作。在 1 秒后，该操作要么成功（调用 resolve），要么失败（调用 reject）。通过 then()、catch() 和 finally() 方法，我们可以处理不同的执行结果。
# 同步问题

## 使用锁和条件变量
如果您有三个需要按顺序依次执行的过程，例如数据加载、数据处理和结果输出，那么使用多个锁和条件变量来协调这些过程的执行是非常有效的方法。这可以确保每个过程在前一个过程完成后才会启动。
以下是如何使用两个锁和两个条件变量来实现这一目的的示例代码：
代码结构：
假设我们有三个函数：load_data（加载数据）、process_data（处理数据）和 output_results（输出结果）。我们希望它们按顺序执行。
import threading

### 创建锁和条件变量
data_ready_lock = threading.Lock()
data_ready_condition = threading.Condition(data_ready_lock)

process_ready_lock = threading.Lock()
process_ready_condition = threading.Condition(process_ready_lock)

# 初始化状态变量
data_loaded = False
data_processed = False

def load_data():
    global data_loaded
    with data_ready_condition:
        print("Loading data...")
        # 模拟数据加载过程
        # ... (数据加载逻辑)
        data_loaded = True
        print("Data loading completed.")
        data_ready_condition.notify_all()  # 通知处理数据的线程

def process_data():
    global data_processed
    with data_ready_condition:
        while not data_loaded:
            data_ready_condition.wait()  # 等待数据加载完成
    with process_ready_condition:
        print("Processing data...")
        # 模拟数据处理过程
        # ... (数据处理逻辑)
        data_processed = True
        print("Data processing completed.")
        process_ready_condition.notify_all()  # 通知结果输出的线程

def output_results():
    with process_ready_condition:
        while not data_processed:
            process_ready_condition.wait()  # 等待数据处理完成
    print("Outputting results...")
    # 模拟结果输出过程
    # ... (结果输出逻辑)
    print("Results output completed.")

# 启动线程来执行这些函数
thread1 = threading.Thread(target=load_data)
thread2 = threading.Thread(target=process_data)
thread3 = threading.Thread(target=output_results)

thread1.start()
thread2.start()
thread3.start()

thread1.join()
thread2.join()
thread3.join()

详细说明：
1. 创建锁和条件变量：
    * data_ready_lock 和 data_ready_condition 用于协调 load_data 和 process_data 的执行。
    * process_ready_lock 和 process_ready_condition 用于协调 process_data 和 output_results 的执行。
2. 状态变量：
    * data_loaded 和 data_processed 用来表示前一个过程是否已经完成。它们的初始值为 False。
3. load_data 函数：
    * 获取 data_ready_condition 的锁后，模拟数据加载过程。
    * 加载完成后，将 data_loaded 设置为 True，并调用 data_ready_condition.notify_all() 通知等待的线程。
4. process_data 函数：
    * 首先等待 data_ready_condition 通知，直到 data_loaded 为 True。
    * 获取 process_ready_condition 的锁，模拟数据处理过程。
    * 处理完成后，将 data_processed 设置为 True，并调用 process_ready_condition.notify_all() 通知等待的线程。
5. output_results 函数：
    * 等待 process_ready_condition 通知，直到 data_processed 为 True，然后进行结果输出。
6. 线程启动和同步：
    * 通过 threading.Thread 启动每个过程。
    * 使用 join 方法确保主线程等待所有子线程完成。
## 使用状态变量和定时检查
要使用状态变量和定时检查来确保四个函数 create_database、load_data、process_data 和 output_results 顺序执行，可以利用状态变量来表示每个阶段是否已经完成。然后在每个函数中，使用一个循环不断检查上一个阶段是否完成，以决定当前阶段是否可以继续执行。
下面是一个示例代码，演示如何实现这一点：
import sqlite3
import threading
import time

# 定义全局状态变量
database_created = False
data_loaded = False
data_processed = False

# 定义数据库路径
DATABASE_PATH = 'example.db'

def create_database():
    global database_created
    # 创建数据库连接
    conn = sqlite3.connect(DATABASE_PATH)
    cursor = conn.cursor()
    try:
        print("Creating database...")
        # 创建表
        cursor.execute('CREATE TABLE IF NOT EXISTS data (id INTEGER PRIMARY KEY, value TEXT)')
        conn.commit()
        database_created = True
        print("Database created.")
    finally:
        conn.close()

def load_data():
    global data_loaded
    while not database_created:
        print("Waiting for database to be created...")
        time.sleep(1)  # 每秒检查一次

    # 创建数据库连接
    conn = sqlite3.connect(DATABASE_PATH)
    cursor = conn.cursor()
    try:
        print("Loading data...")
        # 插入数据
        cursor.execute('INSERT INTO data (value) VALUES ("Sample data")')
        conn.commit()
        data_loaded = True
        print("Data loaded.")
    finally:
        conn.close()

def process_data():
    global data_processed
    while not data_loaded:
        print("Waiting for data to be loaded...")
        time.sleep(1)  # 每秒检查一次

    # 创建数据库连接
    conn = sqlite3.connect(DATABASE_PATH)
    cursor = conn.cursor()
    try:
        print("Processing data...")
        cursor.execute('SELECT * FROM data')
        result = cursor.fetchall()
        # 模拟数据处理
        print(f"Data processed: {result}")
        data_processed = True
        print("Data processing completed.")
    finally:
        conn.close()

def output_results():
    while not data_processed:
        print("Waiting for data to be processed...")
        time.sleep(1)  # 每秒检查一次

    # 创建数据库连接
    conn = sqlite3.connect(DATABASE_PATH)
    cursor = conn.cursor()
    try:
        print("Outputting results...")
        cursor.execute('SELECT * FROM data')
        result = cursor.fetchall()
        # 模拟结果输出
        print(f"Results: {result}")
        print("Results output completed.")
    finally:
        conn.close()

# 启动线程来执行这些函数
thread1 = threading.Thread(target=create_database)
thread2 = threading.Thread(target=load_data)
thread3 = threading.Thread(target=process_data)
thread4 = threading.Thread(target=output_results)

thread1.start()
thread2.start()
thread3.start()
thread4.start()

thread1.join()
thread2.join()
thread3.join()
thread4.join()
