brew install argocd
argocd login 127.0.0.1:36339 --insecure # management 56RlSfaKIvuWyCSf NOK
argocd login 127.0.0.1:8443 --insecure # management 56RlSfaKIvuWyCSf OK
argocd login 127.0.0.1:37803 --insecure # production
argocd login 127.0.0.1:44707 --insecure # staging


argocd login localhost:8083 --insecure 

kxAt1l1jWz7RA5SV

kubectl port-forward pod-name 1234:1234 
kubectl port-forward -n argocd service/argocd-server 8443:443 > /dev/null 2>&1 &
kubectl port-forward svc/argocd-server -n argocd 8083:80 > /dev/null 2>&1 &
