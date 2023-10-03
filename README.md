Project 4 - Build CI/CD Pipelines, Monitoring and Logging - Project: Movie Picture Pipeline
Xử lý Pipeline tự động lint, test, build và deploy backend python app và front end reactjs app lên AWS EKS sử dụng GitHub Action
Nội Dung:
Chạy code và fix lỗi code trên local: Có thể dùng docker, docker compose
Tạo code CI, CD trong folder .github/workflows: backend-ci.yaml, backend-cd.yaml, frontend-ci.yaml, frontend-cd.yaml
Có thể tạo chung 1 project chứ đừng tách thành 2 repository: trong code yaml có thể chỉ định working_directory khi run job steps
Set Github secret variable với thông tin lấy từ access key tạo ở bước Generate AWS access keys for GitHub Actions
Push code lên github để tự chạy CI/CD. Nhớ phải tạo xong resource trước hãy push
Lưu ý khi code CI/CD:
Trước khi build hay deploy backend hay frontend image cần login ECR để lấy thông tin Registry, tạo biến env REGISTRY, REPOSITORY, IMAGE_TAG
Configure AWS credentials: uses: aws-actions/configure-aws-credentials@v4
Login ECR: uses: aws-actions/amazon-ecr-login@v1
Docker build: docker build -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .
Docker Push: docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG
Khi deploy cũng tương tự cần Configure AWS credentials, login ECR để lấy thông tin Registry nạp vào kustomize
run: kustomize edit set image backend=$REGISTRY/$REPOSITORY:$IMAGE_TAG
run: kustomize build | kubectl apply -f -
Khi build image cho frontend cần get Load Balancer External IP để thay vào build-arg REACT_APP_MOVIE_API_URL:
aws eks update-kubeconfig --name cluster
export EXTERNALIP=$(kubectl get svc backend -o=jsonpath='{.status.loadBalancer.ingress[0].hostname}{"\n"}{.status.loadBalancer.ingress[0].ip}')
echo $EXTERNALIP
docker build -t $REGISTRY/$REPOSITORY:$IMAGE_TAG --build-arg=REACT_APP_MOVIE_API_URL=http://$EXTERNALIP
Các lệnh kubectl hay dùng:
kubectl get svc => xem service, external-ip
kubectl get pods => xem pods, check pod running or not
kubectl logs <pod-name> => xem log của backend
kubectl get deployments -o wide => xem image của các deployment
Lỗi có thể gặp:
Python App: Thiếu package flake8, pip install lại
Code terraform có 1 lỗi, tham khảo cách fix tại: Knowledge - Udacity
Configure AWS credentials trong CI/CD dùng Access Key từ github-action-user không cần dùng thông tin từ cloud lab
Tool Version:
Python 11
Terraform 1.3.9