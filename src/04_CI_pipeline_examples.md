**Introduction**:

In this series of examples, we will explore three different approaches to setting up CI/CD pipelines using GitLab's `.gitlab-ci.yml` configuration. We will start with a basic linear pipeline with multiple jobs, then move on to a directed acyclic graph (DAG) pipeline with parallel execution capabilities, and finally, we'll demonstrate how to create an enhanced parent-child pipeline setup for managing complex projects.

1. **Basic Linear Pipeline with Multiple Jobs**:

   ```yaml
   stages:
     - build
     - test
     - deploy

   build_a:
     stage: build
     script:
       - echo "Building component A..."

   build_b:
     stage: build
     script:
       - echo "Building component B..."

   test_a:
     stage: test
     script:
       - echo "Running tests for component A..."

   test_b:
     stage: test
     script:
       - echo "Running tests for component B..."
       - sleep 10

   deploy_a:
     stage: deploy
     script:
       - echo "Deploying component A..."

   deploy_b:
     stage: deploy
     script:
       - echo "Deploying component B..."
   ```

   **Explanation**:
   
   In this basic linear pipeline, multiple jobs (`build_a`, `build_b`, `test_a`, `test_b`, `deploy_a`, and `deploy_b`) execute within each stage (`build`, `test`, `deploy`), showcasing parallel execution within each stage and sequential dependencies between stages.

   **Key Benefits**:
   
   - Simple and straightforward pipeline structure.
   - Clear sequential execution of stages.
   
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

2. **Directed Acyclic Graph (DAG) Pipeline**:

   ```yaml
   stages:
     - build
     - test
     - deploy

   build_a:
     stage: build
     script:
       - echo "Building component A..."

   build_b:
     stage: build
     script:
       - echo "Building component B..."

   test_a:
     stage: test
     script:
       - echo "Running tests for component A..."
     needs:
       - build_a

   test_b:
     stage: test
     script:
       - echo "Running tests for component B..."
       - sleep 10
     needs:
       - build_b

   deploy_a:
     stage: deploy
     script:
       - echo "Deploying component A..."
     needs:
       - test_a

   deploy_b:
     stage: deploy
     script:
       - echo "Deploying component B..."
     needs:
       - test_b
   ```

   **Explanation**:
   
   In this DAG pipeline, job dependencies within each stage are defined using the `needs` keyword, allowing stages A and B to run independently, showcasing parallel execution.

   **Key Benefits**:
   
   - Flexible and efficient parallel execution.
   - Dependencies can be defined for individual jobs.

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

3. **Enhanced Parent-Child Pipeline with Trigger and DAG Tracks**:

   Parent Pipeline (`.gitlab-ci-parent.yml`):

   ```yaml
   stages:
     - triggers

   trigger_a:
     stage: triggers
     trigger:
       include: .gitlab-ci-child-a.yml
     rules:
       - changes:
           - a/*

   trigger_b:
     stage: triggers
     trigger:
       include: .gitlab-ci-child-b.yml
     rules:
       - changes:
           - b/*
   ```

   Child Pipeline for Component A (`.gitlab-ci-child-a.yml`):

   ```yaml
   stages:
     - build
     - test
     - deploy

   build_a:
     stage: build
     script:
       - echo "Parent pipeline: Triggering Child A when changes are made in 'a/'..."
       - echo "Building component A..."

   test_a:
     stage: test
     script:
       - echo "Running tests for component A..."
     needs:
       - build_a

   deploy_a:
     stage: deploy
     script:
       - echo "Deploying component A..."
     needs:
       - test_a
   ```

   Child Pipeline for Component B (`.gitlab-ci-child-b.yml`):

   ```yaml
   stages:
     - build
     - test
     - deploy

   build_b:
     stage: build
     script:
       - echo "Parent pipeline: Triggering Child B when changes are made in 'b/'..."
       - echo "Building component B..."

   test_b:
     stage: test
     script:
       - echo "Running tests for component B..."
       - sleep 10
     needs:
       - build_b

   deploy_b:
     stage: deploy
     script:
       - echo "Deploying component B..."
     needs:
       - test_b
   ```

   **Explanation**:
   
   In this enhanced parent-child pipeline setup, the parent pipeline (`.gitlab-ci-parent.yml`) uses the `include` keyword to include child pipeline configurations (`.gitlab-ci-child-a.yml` and `.gitlab-ci-child-b.yml`). Each child pipeline is triggered when relevant changes occur within the `a/` and `b/` directories, respectively. Each child pipeline can further execute its own DAG-based tasks independently.

   **Key Benefits**:
   
   - Modular and scalable pipeline management.
   - Fine-grained control over triggers and dependencies.

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

**Conclusion**:

In this document, we have explored three different approaches to CI/CD pipelines using GitLab's `.gitlab-ci.yml` configuration. Each approach offers its own advantages and is suited for different scenarios:

- The basic linear pipeline provides simplicity and clear sequential execution.
- The DAG pipeline allows for flexible and efficient parallel execution with individual job dependencies.
- The enhanced parent-child pipeline offers modular and scalable pipeline management with fine-grained control over triggers and dependencies.

The choice of pipeline approach depends on the specific requirements and complexity of your project. Understanding these concepts can help you design effective CI/CD workflows for your development projects.
