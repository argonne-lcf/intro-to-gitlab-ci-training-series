**Introduction**:

In this series of examples, we will explore three different approaches to setting up CI/CD pipelines using GitLab's `.gitlab-ci.yml` configuration. We will start with a basic linear pipeline with multiple jobs, then move on to a directed acyclic graph (DAG) pipeline with parallel execution capabilities, and finally, we'll demonstrate how to create an enhanced parent-child pipeline setup for managing complex projects.

1. **Basic Linear Pipeline with Multiple Jobs**:

   ```mermaid
   graph LR;
   subgraph Stage_Build
     build -->|Parallel| build_a
     build -->|Parallel| build_b
   end
   subgraph Stage_Test
     test -->|Parallel| test_a
     test -->|Parallel| test_b
   end
   subgraph Stage_Deploy
     deploy -->|Parallel| deploy_a
     deploy -->|Parallel| deploy_b
   end
   Stage_Build --> Stage_Test
   Stage_Test --> Stage_Deploy
   ```

   **Explanation**:
   
   In this basic linear pipeline, multiple jobs (`build_a`, `build_b`, `test_a`, `test_b`, `deploy_a`, and `deploy_b`) execute within each stage (`build`, `test`, `deploy`), showcasing parallel execution within each stage and sequential dependencies between stages.

   **Key Benefits**:
   
   - Simple and straightforward pipeline structure.
   - Clear sequential execution of stages.

   ```yaml
   stages:
     - build
     - test
     - deploy

   build_a:
     stage: build
     script:
       - echo "Building component A..."
   ```

2. **Directed Acyclic Graph (DAG) Pipeline**:

   ```mermaid
   graph TD;

   subgraph A_Track[Track A]
     direction LR;
     build_a --> test_a
     test_a --> deploy_a
   end

   subgraph B_Track[Track B]
     direction LR;
     build_b --> test_b
     test_b --> deploy_b
   end

   A_Track ~~~ B_Track
   ```

   **Explanation**:
   
   In this DAG pipeline, job dependencies within each stage are defined using the `needs` keyword, allowing stages A and B to run independently, showcasing parallel execution.

   **Key Benefits**:
   
   - Flexible and efficient parallel execution.
   - Dependencies can be defined for individual jobs.

   ```yaml
   stages:
     - build
     - test
     - deploy

   build_a:
     stage: build
     script:
       - echo "Building component A..."
   ```

3. **Enhanced Parent-Child Pipeline with Trigger and DAG Tracks**:

   ```mermaid
   graph TD;

   subgraph Parent_Pipeline[Parent Pipeline]
     trigger_a --> .gitlab-ci-child-a.yml
     trigger_b --> .gitlab-ci-child-b.yml
   end

   subgraph Child_Pipeline_A[Child Pipeline A]
     direction LR;
     build_a --> test_a
     test_a --> deploy_a
   end

   subgraph Child_Pipeline_B[Child Pipeline B]
     direction LR;
     build_b --> test_b
     test_b --> deploy_b
   end

   .gitlab-ci-child-a.yml .-> Child_Pipeline_A
   .gitlab-ci-child-b.yml .-> Child_Pipeline_B
   ```

   **Explanation**:
   
   In this enhanced parent-child pipeline setup, the parent pipeline (`.gitlab-ci-parent.yml`) uses the `include` keyword to include child pipeline configurations (`.gitlab-ci-child-a.yml` and `.gitlab-ci-child-b.yml`). Each child pipeline is triggered when relevant changes occur within the `a/` and `b/` directories, respectively. Each child pipeline can further execute its own DAG-based tasks independently.

   **Key Benefits**:
   
   - Modular and scalable pipeline management.
   - Fine-grained control over triggers and dependencies.

   ```yaml
   stages:
     - build
     - test
     - deploy

   build_a:
     stage: build
     script:
       - echo "Building component A..."
   ```

**Conclusion**:

In this document, we have explored three different approaches to CI/CD pipelines using GitLab's `.gitlab-ci.yml` configuration. Each approach offers its own advantages and is suited for different scenarios. The choice of pipeline approach depends on the specific requirements and complexity of your project. Understanding these concepts can help you design effective CI/CD workflows for your development projects.

```
