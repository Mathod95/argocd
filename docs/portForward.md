^Z
bg

fg

ps aux | grep kubectl
kill <PID>


kubectl port-forward -n argocd service/argocd-server 8443:443 > /dev/null 2>&1 &
kubectl port-forward svc/argocd-server -n argocd 8083:80 > /dev/null 2>&1 &


kubectl port-forward -n argocd service/argocd-server 8443:443 > /dev/null 2>&1 &
kubectl port-forward -n argocd service/argocd-server 8080:80 > /dev/null 2>&1 &

kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
QFDxM6RjjJAMmUOz





kubectl port-forward -n argocd service/argocd-server 8080:80 > /dev/null 2>&1 &
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
QFDxM6RjjJAMmUOz
argocd login localhost:8083 --insecure
Username: admin
Password: 
'admin:login' logged in successfully
Context 'localhost:8083' updated
~/github/argocd main* ☸ kind-management/default 15:47:40 ❯ 