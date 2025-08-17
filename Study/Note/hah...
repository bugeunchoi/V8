하.. 커밋을 했는데 뭔가 자기 마음대로 저장이 되어있다.. 일단 임시저장


[++ 새벽추가] bt 기능중 한가지 간과한 점이 있다. bt는 모든 코드의 유기적인 흐름을 보여주진 않는다는 것이다. 

#34 0x000055555a173bb3 in v8::internal::compiler::turboshaft::MaglevGraphBuildingPhase::Run (this=<optimized out>, data=0x55555bb459a8, temp_zone=0x55555bb24a00, linkage=0x55555bb25a40) at ../../src/compiler/turboshaft/turbolev-graph-builder.cc:6322

위 라인의 코드는

```
builder.ProcessGraph(maglev_graph);
```

MaglevGraphBuildingPhase::Run 이후에 여러 코드들이 실행되고 위 코드의 함수를 호출하게 된다.

#33 0x000055555a173efa in v8::internal::maglev::GraphProcessor<v8::internal::compiler::turboshaft::NodeProcessorBase, true>::ProcessGraph (this=this@entry=0x7fffffffbf88, graph=graph@entry=0x55555bb5ee48) at ../../src/maglev/maglev-graph-processor.h:134

다음 라인인 이 라인의 코드는

```
node_processor_.PreProcessBasicBlock(block);
```

바로 PreProcessBasicBlock을 실행 하는 것 같지만 사실 우리는 #34에서 부터 ProcessGraph(maglev_graph); 를 타고 왔으므로 

```
class GraphProcessor {
 public:
  template <typename... Args>
  explicit GraphProcessor(Args&&... args)
      : node_processor_(std::forward<Args>(args)...) {}

  void ProcessGraph(Graph* graph) {
    graph_ = graph;

    node_processor_.PreProcessGraph(graph);

    auto process_constants = [&](auto& map) {
      for (auto it = map.begin(); it != map.end();) {
        ProcessResult result =
            node_processor_.Process(it->second, GetCurrentState());
        switch (result) {
          [[likely]] case ProcessResult::kContinue:
            ++it;
            break;
          case ProcessResult::kRemove:
            it = map.erase(it);
            break;
          case ProcessResult::kHoist:
          case ProcessResult::kAbort:
          case ProcessResult::kSkipBlock:
            UNREACHABLE();
        }
      }
    };
    process_constants(graph->constants());
    process_constants(graph->root());
    process_constants(graph->smi());
    process_constants(graph->tagged_index());
    process_constants(graph->int32());
    process_constants(graph->uint32());
    process_constants(graph->intptr());
    process_constants(graph->float64());
    process_constants(graph->external_references());
    process_constants(graph->trusted_constants());

    for (block_it_ = graph->begin(); block_it_ != graph->end(); ++block_it_) {
      BasicBlock* block = *block_it_;

      BlockProcessResult preprocess_result =
          node_processor_.PreProcessBasicBlock(block);
      switch (preprocess_result) {
        [[likely]] case BlockProcessResult::kContinue:
          break;
        case BlockProcessResult::kSkip:
          continue;
      }

... 생략
```

class GraphProcessor의 void ProcessGraph(Graph* graph) 멤버 함수부터 코드를 봐야한다. 어떻게 보면 당연한 말이지만 여기서 유의해야 하는 부분은 bt는 **이미 리턴된 함수는 출력하지 않는다는 점** 이다. 

bt는 단순히 스택 프레임을 기준으로 콜스택만을 출력해줄뿐 중간에 위와 같은 상황이나 콜백함수를 마주치는 등의 과정이 있다면 코드를 절차적으로 이해하는데 어려움이 생긴다.





