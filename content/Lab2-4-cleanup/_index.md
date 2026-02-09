

---

title: "Lab 2 Section IV: Resource Cleanup"
weight: 10
chapter: false

---

# Cleaning Up Your AWS Resources

#### Why It Matters  
Leaving cloud resources running after a lab can incur unexpected charges. In this final step, you’ll learn how to safely remove everything you created—keeping your AWS account tidy and cost‑efficient.

---

## 1. Empty & Delete the S3 Bucket  
1. **S3 → Buckets → webreact2**  
2. **Empty**: click **Empty bucket**, type `permanently delete` to confirm.  
3. **Delete** the bucket when empty.

## 2. Terminate EC2 Instances  
1. **EC2 → Instances**  
2. Select `ServerAI` and `BackendServer` → **Instance state → Terminate instance**.  
3. Confirm termination.

## 3. Delete the RDS Database  
1. **RDS → Databases → LabPostgres**  
2. **Actions → Delete**  
3. Optionally create a final snapshot or skip it for full removal.  
4. Confirm deletion.

## 4. Remove VPC & Related Resources  
1. **VPC → Your VPCs → MyVPC** → **Actions → Delete VPC**  
2. This will cascade‑delete subnets, IGW, route tables, and security groups.  

---

> **Final Tip:** Check **AWS Cost Explorer** next day to confirm no lingering hourly charges remain.
