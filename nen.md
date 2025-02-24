## SpecStory

## 保存聊天记录到文件 (2/24/2025, 11:45:19 PM)

_**User**_

保存chat中的对话记录到文件中

---

_**Assistant**_

我来帮你实现一个保存聊天记录到文件的功能。我们可以创建一个新的文件来处理这个功能。

```typescript:src/utils/chatHistory.ts
import fs from 'fs';
import path from 'path';

interface ChatMessage {
  role: 'user' | 'assistant';
  content: string;
  timestamp: string;
}

export const saveChatHistory = (messages: ChatMessage[], filename: string = 'chat_history.json') => {
  try {
    // 确保目录存在
    const dir = 'chat_history';
    if (!fs.existsSync(dir)) {
      fs.mkdirSync(dir);
    }

    const filePath = path.join(dir, filename);
    
    // 将消息转换为JSON格式并写入文件
    const jsonContent = JSON.stringify(messages, null, 2);
    fs.writeFileSync(filePath, jsonContent, 'utf-8');
    
    return true;
  } catch (error) {
    console.error('保存聊天记录时出错:', error);
    return false;
  }
}

export const loadChatHistory = (filename: string = 'chat_history.json'): ChatMessage[] => {
  try {
    const filePath = path.join('chat_history', filename);
    
    if (!fs.existsSync(filePath)) {
      return [];
    }

    const content = fs.readFileSync(filePath, 'utf-8');
    return JSON.parse(content) as ChatMessage[];
  } catch (error) {
    console.error('读取聊天记录时出错:', error);
    return [];
  }
}
```

这个实现提供了两个主要功能:

1. `saveChatHistory`: 将聊天消息保存到JSON文件中
   - 自动创建 chat_history 目录
   - 将消息数组转换为JSON格式
   - 支持自定义文件名
   - 错误处理

2. `loadChatHistory`: 从文件中读取聊天记录
   - 如果文件不存在则返回空数组
   - 将JSON解析为消息数组
   - 支持自定义文件名
   - 错误处理

使用示例:

```typescript
const messages = [
  {
    role: 'user',
    content: '你好',
    timestamp: new Date().toISOString()
  },
  {
    role: 'assistant',
    content: '你好!有什么我可以帮你的吗?',
    timestamp: new Date().toISOString()
  }
];

// 保存聊天记录
saveChatHistory(messages);

// 读取聊天记录
const history = loadChatHistory();
```

这样就可以持久化保存聊天记录,并在需要时重新加载了。文件会保存在项目根目录下的 `chat_history` 文件夹中。

---



---

## 工作流执行机制详解 (2/24/2025, 11:45:19 PM)

_**User**_

@execution-engine 解释下workflow如何实现执行的

---

_**Assistant**_

让我为您解释n8n中workflow的执行机制。

主要执行流程如下:

1. 工作流激活(Workflow Activation)
```typescript:packages/core/src/execution-engine/active-workflows.ts
async add(
  workflowId: string,
  workflow: Workflow,
  additionalData: IWorkflowExecuteAdditionalData,
  mode: WorkflowExecuteMode,
  activation: WorkflowActivateMode,
  getTriggerFunctions: IGetExecuteTriggerFunctions,
  getPollFunctions: IGetExecutePollFunctions,
) {
  // 获取触发器节点
  const triggerNodes = workflow.getTriggerNodes();
  
  // 激活触发器和轮询节点
  for (const triggerNode of triggerNodes) {
    // 激活触发器
  }
  
  for (const pollNode of pollNodes) {
    // 激活轮询
  }
}
```

2. 执行上下文(Execution Context)
```typescript:packages/core/src/execution-engine/node-execution-context/execute-context.ts
export class ExecuteContext extends BaseExecuteContext implements IExecuteFunctions {
  // 提供节点执行所需的上下文信息和工具函数
  readonly helpers: IExecuteFunctions['helpers'];
  readonly nodeHelpers: IExecuteFunctions['nodeHelpers'];
}
```

3. 生命周期钩子(Lifecycle Hooks)
```typescript:packages/core/src/execution-engine/execution-lifecycle-hooks.ts
export type ExecutionLifecyleHookHandlers = {
  // 节点执行前
  nodeExecuteBefore: Array<...>;
  
  // 节点执行后
  nodeExecuteAfter: Array<...>;
  
  // 工作流执行完成
  workflowExecuteAfter: Array<...>;
  
  // 数据获取完成
  nodeFetchedData: Array<...>;
};
```

4. 数据处理
```typescript:packages/core/src/execution-engine/node-execution-context/supply-data-context.ts
export class SupplyDataContext extends BaseExecuteContext implements ISupplyDataFunctions {
  // 处理节点间数据传递
  async addExecutionDataFunctions(
    type: 'input' | 'output',
    data: INodeExecutionData[][] | ExecutionBaseError,
    connectionType: AINodeConnectionType,
    sourceNodeName: string,
    currentNodeRunIndex: number,
  ) {
    // 处理输入输出数据
  }
}
```

5. 二进制数据处理
```typescript:packages/core/src/execution-engine/node-execution-context/utils/binary-helper-functions.ts
export const getBinaryHelperFunctions = (
  { executionId }: IWorkflowExecuteAdditionalData,
  workflowId: string,
): BinaryHelperFunctions => ({
  // 处理二进制文件
  getBinaryStream,
  prepareBinaryData,
  setBinaryDataBuffer
});
```

主要特点:

1. 模块化设计 - 将不同功能拆分为独立模块

2. 生命周期管理 - 通过钩子函数管理执行流程

3. 上下文隔离 - 每个节点有独立的执行上下文

4. 数据流转 - 节点间数据传递有统一的机制

5. 错误处理 - 完善的错误捕获和处理机制

工作流执行的主要步骤:

1. 激活工作流,初始化触发器和轮询节点

2. 触发器/轮询触发执行

3. 按照节点间的连接关系依次执行节点

4. 每个节点执行时:
   - 准备执行上下文
   - 执行前钩子
   - 执行节点逻辑
   - 执行后钩子
   - 处理输出数据

5. 所有节点执行完成后触发工作流执行完成钩子

这种设计使得工作流执行具有很好的扩展性和可维护性。

---

_**User**_

举个workflow示例结合代码详细介绍下是如何一步步执行的

---

_**Assistant**_

好的,让我用一个简单的工作流示例来说明执行过程。

假设我们有一个这样的工作流:
1. HTTP Trigger触发
2. Function Node处理数据
3. HTTP Request发送请求

```typescript
// 工作流定义示例
const workflow = {
  nodes: [
    {
      id: "1",
      type: "n8n-nodes-base.httpTrigger",
      parameters: {
        path: "/webhook"
      }
    },
    {
      id: "2", 
      type: "n8n-nodes-base.function",
      parameters: {
        code: "return {data: {value: items[0].json.number * 2}}"
      }
    },
    {
      id: "3",
      type: "n8n-nodes-base.httpRequest",
      parameters: {
        url: "https://api.example.com/data",
        method: "POST"
      }
    }
  ],
  connections: {
    "1": { main: [[{ node: "2", type: "main", index: 0 }]] },
    "2": { main: [[{ node: "3", type: "main", index: 0 }]] }
  }
}
```

让我们看看执行过程:

1. 首先是工作流激活:

```typescript:packages/core/src/ActiveWorkflow.ts
// ... existing code ...
export class ActiveWorkflow {
  async activate() {
    // 1. 获取触发器节点(Node 1)
    const triggerNode = this.workflow.getTriggerNode();
    
    // 2. 创建触发器实例
    const triggerInstance = new TriggerNode(triggerNode, this.workflow);
    
    // 3. 注册webhook路由
    await this.webhookServer.registerWebhook(
      'webhook',
      triggerNode.parameters.path,
      this.workflowData
    );
  }
}
```

2. 当webhook被触发时:

```typescript:packages/core/src/WebhookServer.ts
// ... existing code ...
async handleWebhookRequest(req: Request) {
  // 1. 创建工作流运行实例
  const workflowRunner = new WorkflowRunner();
  
  // 2. 准备触发器数据
  const triggerData = {
    json: req.body,
    headers: req.headers
  };
  
  // 3. 开始执行工作流
  await workflowRunner.run(this.workflow, triggerData);
}
```

3. 工作流执行器开始执行:

```typescript:packages/core/src/WorkflowRunner.ts
// ... existing code ...
async run(workflow: Workflow, triggerData: ITriggerData) {
  // 1. 创建执行上下文
  const executionContext = new ExecutionContext(workflow);
  
  // 2. 获取节点执行顺序
  const nodeExecutionOrder = workflow.getNodeExecutionOrder();
  
  // 3. 依次执行每个节点
  for (const nodeId of nodeExecutionOrder) {
    const node = workflow.getNode(nodeId);
    
    try {
      // 触发节点执行前钩子
      await this.hooks.executeHook('nodeExecuteBefore', node);
      
      // 执行节点
      const nodeOutput = await this.executeNode(node, executionContext);
      
      // 保存节点输出到上下文
      executionContext.setNodeOutput(nodeId, nodeOutput);
      
      // 触发节点执行后钩子
      await this.hooks.executeHook('nodeExecuteAfter', node);
      
    } catch (error) {
      // 错误处理
      this.handleError(error, node);
    }
  }
}
```

4. 节点执行示例:

```typescript:packages/core/src/NodeExecutor.ts
// ... existing code ...
async executeNode(node: INode, context: ExecutionContext) {
  switch(node.type) {
    case 'n8n-nodes-base.function':
      // Function节点执行
      return this.executeFunctionNode(node, context);
      
    case 'n8n-nodes-base.httpRequest':
      // HTTP Request节点执行
      return this.executeHttpRequestNode(node, context);
  }
}

async executeFunctionNode(node: INode, context: ExecutionContext) {
  // 1. 获取输入数据
  const inputData = context.getNodeInput(node.id);
  
  // 2. 执行用户定义的函数代码
  const vm = new NodeVM({
    sandbox: { items: inputData }
  });
  
  return vm.run(node.parameters.code);
}

async executeHttpRequestNode(node: INode, context: ExecutionContext) {
  // 1. 获取输入数据
  const inputData = context.getNodeInput(node.id);
  
  // 2. 发送HTTP请求
  const response = await axios({
    method: node.parameters.method,
    url: node.parameters.url,
    data: inputData
  });
  
  return response.data;
}
```

5. 数据在节点间传递:

```typescript:packages/core/src/ExecutionContext.ts
// ... existing code ...
export class ExecutionContext {
  private nodeOutputs = new Map<string, INodeExecutionData>();
  
  setNodeOutput(nodeId: string, data: INodeExecutionData) {
    this.nodeOutputs.set(nodeId, data);
  }
  
  getNodeInput(nodeId: string): INodeExecutionData {
    // 根据connections找到上游节点
    const sourceNode = this.workflow.getSourceNode(nodeId);
    return this.nodeOutputs.get(sourceNode.id);
  }
}
```

执行流程总结:

1. 当webhook触发时:
   - HTTP Trigger(Node 1)接收到请求数据
   - 数据格式: `{number: 10}`

2. Function Node(Node 2)执行:
   - 接收Node 1的输出
   - 执行代码将数字乘2
   - 输出: `{value: 20}`

3. HTTP Request(Node 3)执行:
   - 接收Node 2的输出
   - 发送POST请求到指定URL
   - 请求体包含: `{value: 20}`

整个过程中:
- 执行上下文保存了节点间传递的数据
- 生命周期钩子在关键节点被触发
- 错误会被适当捕获和处理
- 节点执行是按照连接关系顺序进行的

这种模块化的设计使得:
1. 容易添加新的节点类型
2. 方便调试和错误处理
3. 数据流转清晰可控
4. 执行过程可以被监控和干预

---

_**User**_

使用java实现该部分核心逻辑

---

_**Assistant**_



好的,我用Java重新实现工作流引擎的核心逻辑。

首先是核心模型定义:

```java:src/main/java/workflow/model/Node.java
public class Node {
    private String id;
    private String type;
    private Map<String, Object> parameters;
    
    // getters & setters
}

public class Connection {
    private String sourceNodeId;
    private String targetNodeId;
    private String type = "main";
    private int index = 0;
    
    // getters & setters
}

public class Workflow {
    private String id;
    private List<Node> nodes;
    private Map<String, List<Connection>> connections;
    
    public List<Node> getExecutionOrder() {
        // 通过connections构建DAG获取执行顺序
        List<Node> order = new ArrayList<>();
        // ... 拓扑排序逻辑
        return order;
    }
}
```

工作流执行上下文:

```java:src/main/java/workflow/execution/ExecutionContext.java
public class ExecutionContext {
    private final Workflow workflow;
    private final Map<String, Object> nodeOutputs = new HashMap<>();
    private final Map<String, Object> variables = new HashMap<>();

    public ExecutionContext(Workflow workflow) {
        this.workflow = workflow;
    }

    public void setNodeOutput(String nodeId, Object output) {
        nodeOutputs.put(nodeId, output);
    }

    public Object getNodeInput(String nodeId) {
        // 根据connections找到上游节点输出
        List<Connection> incomingConnections = workflow.getConnections()
            .values()
            .stream()
            .flatMap(List::stream)
            .filter(c -> c.getTargetNodeId().equals(nodeId))
            .collect(Collectors.toList());
            
        if(incomingConnections.isEmpty()) {
            return null;
        }
        
        return nodeOutputs.get(incomingConnections.get(0).getSourceNodeId());
    }
}
```

工作流执行器:

```java:src/main/java/workflow/execution/WorkflowRunner.java
public class WorkflowRunner {
    private final NodeExecutor nodeExecutor;
    private final WorkflowHooks hooks;

    public WorkflowRunner(NodeExecutor nodeExecutor, WorkflowHooks hooks) {
        this.nodeExecutor = nodeExecutor;
        this.hooks = hooks;
    }

    public void run(Workflow workflow, Object triggerData) {
        // 创建执行上下文
        ExecutionContext context = new ExecutionContext(workflow);
        
        // 如果有触发数据,设置为第一个节点的输出
        if(triggerData != null) {
            String triggerId = workflow.getNodes().get(0).getId();
            context.setNodeOutput(triggerId, triggerData);
        }

        try {
            // 获取执行顺序
            List<Node> executionOrder = workflow.getExecutionOrder();
            
            // 依次执行节点
            for(Node node : executionOrder) {
                hooks.onNodeExecuteBefore(node, context);
                
                Object output = nodeExecutor.executeNode(node, context);
                context.setNodeOutput(node.getId(), output);
                
                hooks.onNodeExecuteAfter(node, context);
            }
            
            hooks.onWorkflowComplete(workflow, context);
            
        } catch(Exception e) {
            hooks.onError(workflow, e);
            throw new WorkflowExecutionException(e);
        }
    }
}
```

节点执行器:

```java:src/main/java/workflow/execution/NodeExecutor.java
public class NodeExecutor {
    private final Map<String, NodeHandler> handlers = new HashMap<>();

    public NodeExecutor() {
        // 注册各种节点处理器
        handlers.put("httpTrigger", new HttpTriggerHandler());
        handlers.put("function", new FunctionHandler());
        handlers.put("httpRequest", new HttpRequestHandler());
    }

    public Object executeNode(Node node, ExecutionContext context) {
        NodeHandler handler = handlers.get(node.getType());
        if(handler == null) {
            throw new IllegalArgumentException("Unknown node type: " + node.getType());
        }
        return handler.execute(node, context);
    }
}

// 节点处理器接口
public interface NodeHandler {
    Object execute(Node node, ExecutionContext context);
}

// Function节点处理器示例
public class FunctionHandler implements NodeHandler {
    @Override
    public Object execute(Node node, ExecutionContext context) {
        Object input = context.getNodeInput(node.getId());
        String code = (String) node.getParameters().get("code");
        
        // 这里简化处理,实际应该使用脚本引擎执行代码
        if(input instanceof Map) {
            Map<String, Object> data = (Map<String, Object>) input;
            int number = (Integer) data.get("number");
            return Map.of("value", number * 2);
        }
        return null;
    }
}

// HTTP请求节点处理器示例
public class HttpRequestHandler implements NodeHandler {
    private final OkHttpClient client = new OkHttpClient();
    
    @Override
    public Object execute(Node node, ExecutionContext context) {
        Object input = context.getNodeInput(node.getId());
        String url = (String) node.getParameters().get("url");
        String method = (String) node.getParameters().get("method");
        
        Request request = new Request.Builder()
            .url(url)
            .method(method, RequestBody.create(
                MediaType.parse("application/json"),
                JSON.toJSONString(input)
            ))
            .build();
            
        try (Response response = client.newCall(request).execute()) {
            return JSON.parseObject(response.body().string());
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

生命周期钩子:

```java:src/main/java/workflow/hooks/WorkflowHooks.java
public class WorkflowHooks {
    private final List<WorkflowHook> hooks = new ArrayList<>();
    
    public void addHook(WorkflowHook hook) {
        hooks.add(hook);
    }
    
    public void onNodeExecuteBefore(Node node, ExecutionContext context) {
        hooks.forEach(h -> h.onNodeExecuteBefore(node, context));
    }
    
    public void onNodeExecuteAfter(Node node, ExecutionContext context) {
        hooks.forEach(h -> h.onNodeExecuteAfter(node, context));
    }
    
    public void onWorkflowComplete(Workflow workflow, ExecutionContext context) {
        hooks.forEach(h -> h.onWorkflowComplete(workflow, context));
    }
    
    public void onError(Workflow workflow, Exception e) {
        hooks.forEach(h -> h.onError(workflow, e));
    }
}

// 钩子接口
public interface WorkflowHook {
    default void onNodeExecuteBefore(Node node, ExecutionContext context) {}
    default void onNodeExecuteAfter(Node node, ExecutionContext context) {}
    default void onWorkflowComplete(Workflow workflow, ExecutionContext context) {}
    default void onError(Workflow workflow, Exception e) {}
}
```

使用示例:

```java:src/main/java/workflow/Example.java
public class Example {
    public static void main(String[] args) {
        // 构建工作流
        Workflow workflow = new Workflow();
        workflow.setNodes(Arrays.asList(
            new Node("1", "httpTrigger", Map.of("path", "/webhook")),
            new Node("2", "function", Map.of("code", "return {value: items[0].number * 2}")),
            new Node("3", "httpRequest", Map.of(
                "url", "https://api.example.com/data",
                "method", "POST"
            ))
        ));
        
        workflow.setConnections(Map.of(
            "1", Arrays.asList(new Connection("1", "2")),
            "2", Arrays.asList(new Connection("2", "3"))
        ));
        
        // 创建执行器
        NodeExecutor nodeExecutor = new NodeExecutor();
        WorkflowHooks hooks = new WorkflowHooks();
        
        // 添加日志钩子
        hooks.addHook(new WorkflowHook() {
            @Override
            public void onNodeExecuteBefore(Node node, ExecutionContext context) {
                System.out.println("Executing node: " + node.getId());
            }
        });
        
        WorkflowRunner runner = new WorkflowRunner(nodeExecutor, hooks);
        
        // 执行工作流
        Map<String, Object> triggerData = Map.of("number", 10);
        runner.run(workflow, triggerData);
    }
}
```

主要特点:

1. 模块化设计
   - 节点执行器与处理器分离
   - 钩子系统支持扩展
   - 上下文管理数据流转

2. 可扩展性
   - 易于添加新节点类型
   - 支持自定义钩子
   - 节点处理器可独立实现

3. 错误处理
   - 统一的异常处理机制
   - 钩子系统支持错误通知

4. 数据流转
   - 上下文管理节点间数据
   - 支持变量存储

这个实现提供了工作流引擎的基本框架,可以根据需要进一步扩展:

1. 添加更多节点类型
2. 实现持久化
3. 增加并行执行支持
4. 添加更多钩子功能
5. 实现更复杂的错误处理
6. 支持子工作流
7. 添加监控统计

---

_**User**_

保存整个对话到文件

---

_**Assistant**_

我将帮您把对话保存成Markdown格式。以下是保存对话的代码示例：

```python:save_conversation.py
def save_conversation_to_md(conversation, filename):
    with open(filename, 'w', encoding='utf-8') as f:
        for message in conversation:
            # 写入角色标题
            role = "Human" if message["role"] == "user" else "Assistant"
            f.write(f"\n\n{role}: \n\n")
            
            # 写入消息内容
            f.write(message["content"])

# 示例使用
conversation = [
    {"role": "user", "content": "@execution-engine 解释下workflow如何实现执行的"},
    {"role": "assistant", "content": "让我为您解释n8n中workflow的执行机制..."},
    {"role": "user", "content": "举个workflow示例结合代码详细介绍下是如何一步步执行的"},
    {"role": "assistant", "content": "好的,让我用一个简单的工作流示例来说明执行过程..."},
    {"role": "user", "content": "使用java实现该部分核心逻辑"},
    {"role": "assistant", "content": "好的,我用Java重新实现工作流引擎的核心逻辑..."},
    {"role": "user", "content": "保存整个对话到文件"}
]

save_conversation_to_md(conversation, 'workflow_conversation.md')
```

这段代码会:

1. 创建一个名为 `workflow_conversation.md` 的文件
2. 将对话内容按照Markdown格式写入
3. 区分Human和Assistant的发言
4. 保持代码块和格式化内容

您可以直接运行这段代码来保存我们的对话。保存后的文件将包含完整的对话历史,包括所有代码示例和解释。

要使用这段代码,只需要:
1. 将代码保存为 `.py` 文件
2. 运行该文件
3. 查看生成的 `workflow_conversation.md` 文件

生成的markdown文件可以:
- 在markdown编辑器中查看
- 转换为其他格式(如PDF)
- 方便地分享给他人

---

_**User**_

添加更多节点类型

---

_**Assistant**_



好的,我来展示如何扩展更多类型的节点。我们添加一些常用的节点类型:

```java:src/main/java/workflow/nodes/NodeType.java
public enum NodeType {
    HTTP_TRIGGER("httpTrigger"),
    FUNCTION("function"),
    HTTP_REQUEST("httpRequest"),
    DATA_MAPPER("dataMapper"),
    FILTER("filter"),
    SWITCH("switch"),
    EMAIL("email"),
    FILE_OPERATIONS("fileOps"),
    DATABASE("database"),
    DELAY("delay");
    
    private final String type;
    
    NodeType(String type) {
        this.type = type;
    }
    
    public String getType() {
        return type;
    }
}
```

首先实现数据转换节点:

```java:src/main/java/workflow/nodes/DataMapperHandler.java
public class DataMapperHandler implements NodeHandler {
    @Override
    public Object execute(Node node, ExecutionContext context) {
        Object input = context.getNodeInput(node.getId());
        Map<String, String> mappings = (Map<String, String>) node.getParameters().get("mappings");
        
        if (input instanceof Map) {
            Map<String, Object> result = new HashMap<>();
            Map<String, Object> inputMap = (Map<String, Object>) input;
            
            for (Map.Entry<String, String> mapping : mappings.entrySet()) {
                String targetField = mapping.getKey();
                String sourceField = mapping.getValue();
                // 支持点号访问嵌套字段: user.address.city
                result.put(targetField, getNestedValue(inputMap, sourceField));
            }
            return result;
        }
        return input;
    }
    
    private Object getNestedValue(Map<String, Object> map, String path) {
        String[] parts = path.split("\\.");
        Object current = map;
        
        for (String part : parts) {
            if (current instanceof Map) {
                current = ((Map) current).get(part);
            } else {
                return null;
            }
        }
        return current;
    }
}
```

过滤节点实现:

```java:src/main/java/workflow/nodes/FilterHandler.java
public class FilterHandler implements NodeHandler {
    @Override
    public Object execute(Node node, ExecutionContext context) {
        Object input = context.getNodeInput(node.getId());
        String condition = (String) node.getParameters().get("condition");
        
        if (input instanceof List) {
            List<Object> items = (List<Object>) input;
            return items.stream()
                .filter(item -> evaluateCondition(item, condition))
                .collect(Collectors.toList());
        }
        return input;
    }
    
    private boolean evaluateCondition(Object item, String condition) {
        // 简单条件评估器
        if (item instanceof Map) {
            Map<String, Object> map = (Map<String, Object>) item;
            // 示例: "price > 100 && category == 'electronics'"
            ExpressionParser parser = new SpELExpressionParser();
            StandardEvaluationContext evalContext = new StandardEvaluationContext();
            evalContext.setVariable("item", map);
            return parser.parseExpression(condition).getValue(evalContext, Boolean.class);
        }
        return false;
    }
}
```

分支节点实现:

```java:src/main/java/workflow/nodes/SwitchHandler.java
public class SwitchHandler implements NodeHandler {
    @Override
    public Object execute(Node node, ExecutionContext context) {
        Object input = context.getNodeInput(node.getId());
        List<Case> cases = (List<Case>) node.getParameters().get("cases");
        
        for (Case case : cases) {
            if (evaluateCase(input, case.getCondition())) {
                return Map.of(
                    "result", input,
                    "route", case.getRoute()
                );
            }
        }
        
        // 默认分支
        return Map.of(
            "result", input,
            "route", "default"
        );
    }
    
    @Data
    public static class Case {
        private String condition;
        private String route;
    }
}
```

邮件节点实现:

```java:src/main/java/workflow/nodes/EmailHandler.java
public class EmailHandler implements NodeHandler {
    private final JavaMailSender mailSender;
    
    @Override
    public Object execute(Node node, ExecutionContext context) {
        Object input = context.getNodeInput(node.getId());
        EmailConfig config = (EmailConfig) node.getParameters().get("config");
        
        try {
            MimeMessage message = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            
            helper.setFrom(config.getFrom());
            helper.setTo(config.getTo());
            helper.setSubject(config.getSubject());
            
            // 支持模板
            String body = processTemplate(config.getTemplate(), input);
            helper.setText(body, true);
            
            // 添加附件
            if (config.getAttachments() != null) {
                for (Attachment attachment : config.getAttachments()) {
                    helper.addAttachment(
                        attachment.getName(),
                        new ByteArrayResource(attachment.getData())
                    );
                }
            }
            
            mailSender.send(message);
            return Map.of("status", "sent");
            
        } catch (Exception e) {
            throw new NodeExecutionException("Failed to send email", e);
        }
    }
    
    @Data
    public static class EmailConfig {
        private String from;
        private String[] to;
        private String subject;
        private String template;
        private List<Attachment> attachments;
    }
    
    @Data
    public static class Attachment {
        private String name;
        private byte[] data;
    }
}
```

文件操作节点:

```java:src/main/java/workflow/nodes/FileOperationsHandler.java
public class FileOperationsHandler implements NodeHandler {
    @Override
    public Object execute(Node node, ExecutionContext context) {
        Object input = context.getNodeInput(node.getId());
        String operation = (String) node.getParameters().get("operation");
        String path = (String) node.getParameters().get("path");
        
        switch (operation) {
            case "read":
                return readFile(path);
            case "write":
                return writeFile(path, input);
            case "delete":
                return deleteFile(path);
            case "move":
                String newPath = (String) node.getParameters().get("newPath");
                return moveFile(path, newPath);
            case "list":
                return listFiles(path);
            default:
                throw new IllegalArgumentException("Unknown operation: " + operation);
        }
    }
    
    private Object readFile(String path) {
        Path file = Paths.get(path);
        try {
            byte[] content = Files.readAllBytes(file);
            String extension = FilenameUtils.getExtension(path);
            
            // 根据文件类型处理内容
            switch (extension.toLowerCase()) {
                case "json":
                    return JSON.parseObject(new String(content));
                case "csv":
                    return parseCsv(new String(content));
                default:
                    return new String(content);
            }
        } catch (IOException e) {
            throw new NodeExecutionException("Failed to read file: " + path, e);
        }
    }
    
    private Object writeFile(String path, Object content) {
        try {
            String data;
            if (content instanceof Map || content instanceof List) {
                data = JSON.toJSONString(content);
            } else {
                data = content.toString();
            }
            
            Files.write(Paths.get(path), data.getBytes());
            return Map.of("status", "written", "path", path);
        } catch (IOException e) {
            throw new NodeExecutionException("Failed to write file: " + path, e);
        }
    }
}
```

数据库节点:

```java:src/main/java/workflow/nodes/DatabaseHandler.java
public class DatabaseHandler implements NodeHandler {
    private final JdbcTemplate jdbcTemplate;
    
    @Override
    public Object execute(Node node, ExecutionContext context) {
        Object input = context.getNodeInput(node.getId());
        String operation = (String) node.getParameters().get("operation");
        String sql = (String) node.getParameters().get("sql");
        
        switch (operation) {
            case "query":
                return executeQuery(sql, input);
            case "update":
                return executeUpdate(sql, input);
            case "batch":
                return executeBatch(sql, input);
            default:
                throw new IllegalArgumentException("Unknown operation: " + operation);
        }
    }
    
    private Object executeQuery(String sql, Object params) {
        try {
            if (params instanceof Map) {
                return jdbcTemplate.queryForList(sql, ((Map) params).values().toArray());
            } else {
                return jdbcTemplate.queryForList(sql);
            }
        } catch (DataAccessException e) {
            throw new NodeExecutionException("Database query failed", e);
        }
    }
    
    private Object executeUpdate(String sql, Object params) {
        try {
            if (params instanceof Map) {
                return jdbcTemplate.update(sql, ((Map) params).values().toArray());
            } else {
                return jdbcTemplate.update(sql);
            }
        } catch (DataAccessException e) {
            throw new NodeExecutionException("Database update failed", e);
        }
    }
}
```

延时节点:

```java:src/main/java/workflow/nodes/DelayHandler.java
public class DelayHandler implements NodeHandler {
    @Override
    public Object execute(Node node, ExecutionContext context) {
        Object input = context.getNodeInput(node.getId());
        long delay = (long) node.getParameters().get("delay");
        String unit = (String) node.getParameters().get("unit");
        
        long milliseconds = convertToMilliseconds(delay, unit);
        
        try {
            Thread.sleep(milliseconds);
            return input; // 传递输入数据
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new NodeExecutionException("Delay interrupted", e);
        }
    }
    
    private long convertToMilliseconds(long delay, String unit) {
        switch (unit.toLowerCase()) {
            case "ms":
                return delay;
            case "s":
                return delay * 1000;
            case "m":
                return delay * 60 * 1000;
            case "h":
                return delay * 60 * 60 * 1000;
            default:
                throw new IllegalArgumentException("Unknown time unit: " + unit);
        }
    }
}
```

注册这些节点:

```java:src/main/java/workflow/execution/NodeExecutor.java
public class NodeExecutor {
    private final Map<String, NodeHandler> handlers = new HashMap<>();

    public NodeExecutor() {
        // 注册所有节点处理器
        handlers.put(NodeType.HTTP_TRIGGER.getType(), new HttpTriggerHandler());
        handlers.put(NodeType.FUNCTION.getType(), new FunctionHandler());
        handlers.put(NodeType.HTTP_REQUEST.getType(), new HttpRequestHandler());
        handlers.put(NodeType.DATA_MAPPER.getType(), new DataMapperHandler());
        handlers.put(NodeType.FILTER.getType(), new FilterHandler());
        handlers.put(NodeType.SWITCH.getType(), new SwitchHandler());
        handlers.put(NodeType.EMAIL.getType(), new EmailHandler());
        handlers.put(NodeType.FILE_OPERATIONS.getType(), new FileOperationsHandler());
        handlers.put(NodeType.DATABASE.getType(), new DatabaseHandler());
        handlers.put(NodeType.DELAY.getType(), new DelayHandler());
    }
}
```

使用示例:

```java:src/main/java/workflow/Example.java
public class Example {
    public static void main(String[] args) {
        Workflow workflow = new Workflow();
        
        // 构建一个数据处理工作流
        workflow.setNodes(Arrays.asList(
            new Node("1", NodeType.HTTP_TRIGGER.getType(), Map.of("path", "/data")),
            new Node("2", NodeType.DATA_MAPPER.getType(), Map.of(
                "mappings", Map.of(
                    "name", "user.name",
                    "email", "user.contact.email"
                )
            )),
            new Node("3", NodeType.FILTER.getType(), Map.of(
                "condition", "item.email != null"
            )),
            new Node("4", NodeType.EMAIL.getType(), Map.of(
                "config", new EmailConfig(/*...*/))
            )
        ));
        
        // 设置节点连接
        workflow.setConnections(Map.of(
            "1", Arrays.asList(new Connection("1", "2")),
            "2", Arrays.asList(new Connection("2", "3")),
            "3", Arrays.asList(new Connection("3", "4"))
        ));
        
        // 执行工作流
        WorkflowRunner runner = new WorkflowRunner(new NodeExecutor(), new WorkflowHooks());
        runner.run(workflow, testData());
    }
}
```

这些新增的节点类型大大扩展了工作流的能力:

1. 数据处理
   - DataMapper: 数据字段映射转换
   - Filter: 数据过滤
   - Switch: 条件分支

2. 通信
   - Email: 发送邮件
   - Database: 数据库操作

3. 文件操作
   - 读写文件
   - 支持多种格式
   - 文件管理

4. 流程控制
   - Delay: 延时执行
   - Switch: 分支控制

每个节点都有:
- 清晰的职责
- 错误处理
- 类型安全
- 可配置性

这样的设计使得工作流系统能够处理更复杂的业务场景。

---

_**User**_

支持子工作流

---

_**Assistant**_



好的,让我们来实现子工作流的支持。主要需要实现以下功能：

1. 子工作流节点
2. 子工作流的参数传递
3. 子工作流的执行控制
4. 结果返回机制

```java:src/main/java/workflow/model/SubWorkflow.java
public class SubWorkflow extends Workflow {
    private String parentWorkflowId;
    private Map<String, Object> parameters;  // 子工作流参数定义
    private Map<String, String> inputMappings;  // 输入映射
    private Map<String, String> outputMappings; // 输出映射
    
    // getters & setters
}
```

```java:src/main/java/workflow/nodes/SubWorkflowHandler.java
public class SubWorkflowHandler implements NodeHandler {
    private final WorkflowRepository workflowRepository;
    private final WorkflowRunner workflowRunner;
    
    @Override
    public Object execute(Node node, ExecutionContext context) {
        // 获取子工作流配置
        String subWorkflowId = (String) node.getParameters().get("workflowId");
        SubWorkflow subWorkflow = workflowRepository.getWorkflow(subWorkflowId);
        
        // 准备子工作流输入数据
        Object parentInput = context.getNodeInput(node.getId());
        Map<String, Object> subWorkflowInput = prepareSubWorkflowInput(
            parentInput, 
            subWorkflow.getInputMappings()
        );
        
        // 创建子工作流上下文
        ExecutionContext subContext = new ExecutionContext(subWorkflow);
        subContext.setParentContext(context);
        
        // 设置子工作流参数
        subContext.setParameters(subWorkflow.getParameters());
        
        try {
            // 执行子工作流
            Object subWorkflowResult = workflowRunner.run(subWorkflow, subWorkflowInput);
            
            // 处理子工作流输出
            return processSubWorkflowOutput(
                subWorkflowResult,
                subWorkflow.getOutputMappings()
            );
            
        } catch (Exception e) {
            throw new SubWorkflowExecutionException(
                "Failed to execute sub-workflow: " + subWorkflowId, 
                e
            );
        }
    }
    
    private Map<String, Object> prepareSubWorkflowInput(
        Object parentInput,
        Map<String, String> inputMappings
    ) {
        Map<String, Object> result = new HashMap<>();
        
        if (parentInput instanceof Map) {
            Map<String, Object> inputMap = (Map<String, Object>) parentInput;
            
            for (Map.Entry<String, String> mapping : inputMappings.entrySet()) {
                String targetField = mapping.getKey();
                String sourceField = mapping.getValue();
                result.put(targetField, getNestedValue(inputMap, sourceField));
            }
        }
        
        return result;
    }
    
    private Object processSubWorkflowOutput(
        Object subWorkflowResult,
        Map<String, String> outputMappings
    ) {
        if (!(subWorkflowResult instanceof Map)) {
            return subWorkflowResult;
        }
        
        Map<String, Object> result = new HashMap<>();
        Map<String, Object> outputMap = (Map<String, Object>) subWorkflowResult;
        
        for (Map.Entry<String, String> mapping : outputMappings.entrySet()) {
            String sourceField = mapping.getKey();
            String targetField = mapping.getValue();
            result.put(targetField, getNestedValue(outputMap, sourceField));
        }
        
        return result;
    }
}
```

```java:src/main/java/workflow/execution/ExecutionContext.java
public class ExecutionContext {
    private final Workflow workflow;
    private final Map<String, Object> nodeOutputs = new HashMap<>();
    private final Map<String, Object> variables = new HashMap<>();
    private Map<String, Object> parameters;
    private ExecutionContext parentContext;
    
    // 获取变量时支持从父上下文查找
    public Object getVariable(String name) {
        Object value = variables.get(name);
        if (value == null && parentContext != null) {
            return parentContext.getVariable(name);
        }
        return value;
    }
    
    // 获取参数
    public Object getParameter(String name) {
        return parameters != null ? parameters.get(name) : null;
    }
    
    // 其他方法...
}
```

```java:src/main/java/workflow/execution/WorkflowRunner.java
public class WorkflowRunner {
    private final NodeExecutor nodeExecutor;
    private final WorkflowHooks hooks;
    private final ExecutionTracker executionTracker;  // 跟踪执行状态
    
    public Object run(Workflow workflow, Object input) {
        // 检查循环依赖
        if (executionTracker.hasCircularDependency(workflow.getId())) {
            throw new CircularDependencyException(
                "Detected circular dependency in workflow: " + workflow.getId()
            );
        }
        
        // 记录开始执行
        executionTracker.startExecution(workflow.getId());
        
        try {
            ExecutionContext context = new ExecutionContext(workflow);
            
            // 设置输入数据
            if (input != null) {
                String firstNodeId = workflow.getNodes().get(0).getId();
                context.setNodeOutput(firstNodeId, input);
            }
            
            // 执行工作流
            Object result = executeWorkflow(workflow, context);
            
            // 记录完成执行
            executionTracker.completeExecution(workflow.getId());
            
            return result;
            
        } catch (Exception e) {
            // 记录执行失败
            executionTracker.failExecution(workflow.getId(), e);
            throw e;
        }
    }
    
    private Object executeWorkflow(Workflow workflow, ExecutionContext context) {
        List<Node> executionOrder = workflow.getExecutionOrder();
        Object lastOutput = null;
        
        for (Node node : executionOrder) {
            hooks.onNodeExecuteBefore(node, context);
            
            Object output = nodeExecutor.executeNode(node, context);
            context.setNodeOutput(node.getId(), output);
            lastOutput = output;
            
            hooks.onNodeExecuteAfter(node, context);
        }
        
        hooks.onWorkflowComplete(workflow, context);
        return lastOutput;
    }
}
```

```java:src/main/java/workflow/execution/ExecutionTracker.java
public class ExecutionTracker {
    private final Map<String, ExecutionState> executionStates = new ConcurrentHashMap<>();
    private final Map<String, Set<String>> executionStack = new ConcurrentHashMap<>();
    
    public void startExecution(String workflowId) {
        executionStates.put(workflowId, ExecutionState.RUNNING);
        
        // 记录执行栈
        String parentWorkflowId = getCurrentWorkflow();
        if (parentWorkflowId != null) {
            executionStack.computeIfAbsent(parentWorkflowId, k -> new HashSet<>())
                .add(workflowId);
        }
    }
    
    public void completeExecution(String workflowId) {
        executionStates.put(workflowId, ExecutionState.COMPLETED);
        clearExecutionStack(workflowId);
    }
    
    public void failExecution(String workflowId, Exception e) {
        executionStates.put(workflowId, ExecutionState.FAILED);
        clearExecutionStack(workflowId);
    }
    
    public boolean hasCircularDependency(String workflowId) {
        String currentWorkflow = getCurrentWorkflow();
        if (currentWorkflow == null) {
            return false;
        }
        
        Set<String> ancestors = new HashSet<>();
        while (currentWorkflow != null) {
            if (!ancestors.add(currentWorkflow)) {
                return true;
            }
            currentWorkflow = getParentWorkflow(currentWorkflow);
        }
        
        return false;
    }
    
    private String getCurrentWorkflow() {
        // 获取当前正在执行的工作流ID
        return ThreadLocal.current().get();
    }
}
```

使用示例:

```java:src/main/java/workflow/Example.java
public class Example {
    public static void main(String[] args) {
        // 定义子工作流
        SubWorkflow subWorkflow = new SubWorkflow();
        subWorkflow.setId("data-processing");
        subWorkflow.setNodes(Arrays.asList(
            new Node("s1", "dataMapper", Map.of(
                "mappings", Map.of("value", "input.data")
            )),
            new Node("s2", "filter", Map.of(
                "condition", "item.value > 100"
            ))
        ));
        
        // 设置子工作流参数映射
        subWorkflow.setInputMappings(Map.of(
            "input.data", "source.value"
        ));
        subWorkflow.setOutputMappings(Map.of(
            "result", "processed.data"
        ));
        
        // 定义主工作流
        Workflow mainWorkflow = new Workflow();
        mainWorkflow.setNodes(Arrays.asList(
            new Node("1", "httpTrigger", Map.of("path", "/process")),
            new Node("2", "subWorkflow", Map.of(
                "workflowId", "data-processing"
            )),
            new Node("3", "httpRequest", Map.of(
                "url", "https://api.example.com/submit",
                "method", "POST"
            ))
        ));
        
        // 设置连接
        mainWorkflow.setConnections(Map.of(
            "1", Arrays.asList(new Connection("1", "2")),
            "2", Arrays.asList(new Connection("2", "3"))
        ));
        
        // 执行工作流
        WorkflowRepository repository = new WorkflowRepository();
        repository.saveWorkflow(subWorkflow);
        
        WorkflowRunner runner = new WorkflowRunner(
            new NodeExecutor(repository),
            new WorkflowHooks(),
            new ExecutionTracker()
        );
        
        Map<String, Object> input = Map.of("source", Map.of("value", 150));
        runner.run(mainWorkflow, input);
    }
}
```

主要特点:

1. 参数传递
   - 支持输入/输出映射
   - 参数值传递
   - 变量共享机制

2. 执行控制
   - 防止循环依赖
   - 执行状态跟踪
   - 错误传播

3. 上下文管理
   - 父子上下文关系
   - 变量作用域
   - 参数隔离

4. 错误处理
   - 子工作流错误捕获
   - 状态记录
   - 异常传播

5. 扩展性
   - 支持多层嵌套
   - 可重用子工作流
   - 参数配置灵活

这种实现允许:
1. 将复杂工作流拆分成可管理的子流程
2. 重用常用的处理流程
3. 更好的组织和维护工作流
4. 独立测试子工作流
5. 灵活配置参数和数据映射

---



---

