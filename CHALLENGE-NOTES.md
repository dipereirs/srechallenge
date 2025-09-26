# 🚀 SRE Challenge - Notes

## 📋 Overview
This document detailed notes about my implementation of the challenge, including problems faced, solutions implemented, and lessons learned.

**Total Time Invested:** ~8 hours  
**Completion Status:** ✅ Fully completed with minor certificate validation issues

---

## 🚧 Challenges Encountered & Solutions

### 1. 🐳 Kubernetes Environment Issues
**Problem:** Ingress controller hanging when using Minikube  
**Solution:** Switched to Docker Desktop Kubernetes client  
**Impact:** Resolved deployment issues and improved local development experience

### 2. 🔒 Certificate Validation Challenges
**Problem:** Let's Encrypt certificate validation failing for `dumbkv.example.com`  
**Root Cause:** ACME server refuses to issue certificates for example domains  
**Workaround:** 
- Configured staging environment for testing
- Used Chrome's `thisisunsafe` bypass for local validation
- Application remains fully functional despite certificate warnings

**Technical Details:**
```
Error: 400 urn:ietf:params:acme:error:rejectedIdentifier: 
Cannot issue for "dumbkv.example.com": The ACME server refuses to issue 
a certificate for this domain name, because it is forbidden by policy
```

### 3. 📊 ServiceMonitor Configuration
**Problem:** Prometheus not discovering ServiceMonitor  
**Solution:** Added required `release: prometheus` label to ServiceMonitor  
**Result:** Metrics collection working correctly

---

## 🎯 Key Learnings

1. **Environment Selection Matters** - Docker Desktop Kubernetes proved more reliable than Minikube for this use case
2. **Certificate Management** - Let's Encrypt has strict policies for example domains
3. **Monitoring Integration** - Proper labeling
