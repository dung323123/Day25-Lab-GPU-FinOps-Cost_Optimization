# GPU FinOps Lab Report

## 1. Giới thiệu

- Họ và tên: **Nguyen Manh Dung**
- MSSV: **2A202600177**
- Notebook: `notebook/gpu_finops_lab.ipynb`
- Mục tiêu lab:
  - Theo dõi hạ tầng GPU và workload.
  - Đo lường chi phí theo workload, spot savings, waste.
  - Thử nghiệm tối ưu chi phí bằng autoscaling, spot, AMP.
  - Xây dựng phân tích nâng cao ở Part 8.5 (scaling, forecast, roadmap).

GPU FinOps trong bài lab tập trung vào việc cân bằng giữa hiệu năng huấn luyện và chi phí vận hành, dựa trên dữ liệu runtime thực tế và các chính sách tối ưu.

## 2. Phân tích kết quả

### 2.1 Part 1-7 (Mock Cluster FinOps)

**Part 1 - Cluster Monitoring**
- Cluster gồm **4 nodes / 8 GPUs**.
- Trạng thái ban đầu: **Busy 0, Idle 8, Avg Utilization 10.7%**.
- Tổng công suất tiêu thụ quan sát: **237W**.

**Part 2 - Workload & Billing**
- Submit 4 workload thành công, trạng thái sau submit: **Busy 5/8, Utilization 56.4%**.
- Billing tổng hợp:
  - **Total Cost: $1.1949**
  - **Total Savings: $1.2927**
  - **Budget Used: 1.2%**, Alert: **OK**

**Part 3 - Spot Instance**
- Spot pricing cho thấy discount thực tế:
  - T4: **29.9%**
  - A100: **27.8%**
  - V100: **43.6%**
- Mô phỏng preemption:
  - Preempted: **1 instance**
  - Spot savings report: **$0.0358 (70.0%)**

**Part 4 - Autoscaling**
- Policy được đổi sang ngưỡng scale-up 70%, scale-down 25%.
- Ở mức utilization 56.4%, autoscaler giữ trạng thái **NO_ACTION** (phù hợp ngưỡng).

**Part 5 - Cost Analysis**
- 5 snapshot có waste ổn định: **23.2%**.
- Waste report:
  - **Total Cost: $0.190280**
  - **Total Idle Cost: $0.044165**
  - **Potential Monthly Save: $2289.51**
  - Severity: **LOW**
- Recommendations chính:
  - **USE_SPOT** (ước tính savings ~65%)
  - **SCHEDULING** (ước tính savings ~20%)

**Part 6 - Visualization**
- Đã tạo các biểu đồ:
  - `generated_charts/finops_cost_breakdown.png`
  - `generated_charts/finops_timeseries.png`

**Part 7 - Full Workflow**
- Khi tăng tải: utilization lên **78.8%**, autoscaler quyết định **scale_up**.
- Sau áp dụng spot optimization:
  - Spot savings: **$0.8141 (70.0%)**
  - Final spend: **$1.3640**
  - Final saved: **$1.4151**
  - Budget used: **1.4%**

### 2.2 Part 8 (Real GPU Training - Kaggle/Colab)

**Môi trường GPU thực tế**
- GPU detect: **Tesla T4 (15.6 GB)**
- Giá dùng tính cost: **$0.35/giờ**

**So sánh FP32 vs AMP**

| Metric | FP32 | AMP | Kết quả |
|---|---:|---:|---:|
| Total Time (s) | 123.2 | 59.2 | **2.08x nhanh hơn** |
| Peak Memory (GB) | 0.82 | 0.60 | **tiết kiệm 0.22 GB** |
| Cost (USD) | 0.011975 | 0.005760 | **tiết kiệm $0.006215** |
| Cost Saving | - | - | **51.9%** |
| Avg GPU Util (%) | 94.8 | 90.3 | giảm nhẹ |
| Avg Power (W) | 66.7 | 64.0 | giảm nhẹ |

**Chi phí ở quy mô lớn (extrapolated)**
- 1 ngày: tiết kiệm **$4.36**
- 1 tuần: tiết kiệm **$30.52**
- 1 tháng: tiết kiệm **$130.79**

Kết luận Part 8: AMP cho hiệu quả FinOps rõ rệt, giảm mạnh thời gian và cost trong khi chất lượng huấn luyện vẫn ở mức chấp nhận được.

### 2.3 Part 8.5 (Advanced GPU Cost Optimization)

**Exercise 8.5.1 - Multi-GPU Cost Analysis (A100, baseline 2h)**

| GPU count | Speedup | Scaling efficiency | Time (h) | Total cost (USD) | Cost/speedup |
|---:|---:|---:|---:|---:|---:|
| 1 | 1.000 | 100.0% | 2.000 | 7.34 | 7.34 |
| 2 | 1.852 | 92.6% | 1.080 | 7.93 | 4.28 |
| 4 | 3.226 | 80.6% | 0.620 | 9.10 | 2.82 |
| 8 | 5.128 | 64.1% | 0.390 | 11.45 | 2.23 |

- Chi phí tuyệt đối thấp nhất: **1 GPU ($7.34)**.
- Cost/performance tốt nhất: **8 GPUs** (chi phí cho mỗi đơn vị speedup thấp nhất).

**Exercise 8.5.2 - Project Cost Forecasting**
- Base total cost: **$3551.20**
- Contingency 20%: **+$710.24**
- Expected total: **$4261.44**
- 95% CI: **$2913.12 → $5609.76**

Ý nghĩa: nếu xét bất định, dự án có thể vượt mốc $5000 ở biên trên, do đó cần tối ưu sớm từ đầu.

**Exercise 8.5.3 - Optimization Opportunity Analysis**
- Current cost: **$1468.00**
- Projected final cost: **$179.68**
- Projected savings: **$1288.32 (87.8%)**

Roadmap ưu tiên tập trung vào quick wins (AMP, batch size), sau đó các thay đổi effort cao hơn để tối đa hóa savings.

**Exercise 8.5.4 - Integrated Dashboard**
- Dashboard tổng hợp đã tạo: `generated_charts/advanced_finops_dashboard.png`.
- Dashboard thể hiện đồng thời:
  - Cost curve theo số GPU.
  - Hiệu suất scaling.
  - Forecast + confidence interval.
  - Breakdown chi phí theo phase.
  - Ma trận ưu tiên optimization.
  - Cumulative savings roadmap.

**Exercise 8.5.5 - Challenge Strategy**
- Baseline scenario: **8x A100, 200h = $5872.00**, vượt budget $5000.
- Phương án tối ưu chọn:
  - GPU count khả thi theo deadline: **4 GPUs**.
  - Runtime ước tính: **62.00h**.
  - Cost trước optimization: **$910.16**.
- Sau khi áp dụng chiến lược (risk <= MEDIUM):
  - Projected optimization savings: **$631.65 (69.4%)**
  - Final estimated cost (kèm contingency): **$326.20**
  - Budget check: **PASS**
  - Deadline/risk check: **PASS**

## 3. Kết luận và học hỏi

- Kỹ năng đạt được:
  - Đọc metrics cụm GPU và liên hệ trực tiếp tới chi phí.
  - Theo dõi billing theo workload và phân tích waste.
  - Ứng dụng spot/autoscaling/AMP để giảm cost có kiểm soát.
  - Lập forecast có uncertainty và contingency cho kế hoạch ngân sách.
  - Xây roadmap ưu tiên optimization theo savings, effort, risk.

- Chiến lược hiệu quả nhất trong bài:
  - **AMP** là quick win mạnh, giảm thời gian và cost rõ rệt.
  - **Spot instances** cho mức tiết kiệm cao, nhưng cần quản trị rủi ro preemption.
  - **Kết hợp nhiều chiến lược theo lộ trình** giúp savings tích lũy lớn hơn tối ưu đơn lẻ.

- Ứng dụng thực tế:
  - Có thể dùng pipeline này để kiểm soát chi phí training/inference theo tuần/tháng.
  - Dashboard Part 8.5 phù hợp làm artifact báo cáo FinOps cho team kỹ thuật + tài chính.

## 4. Danh mục minh chứng

**Screenshots chính**
- `screenshots/part1_cluster_monitoring.png`
- `screenshots/part2_workload_submission.png`
- `screenshots/part2_billing_summary.png`
- `screenshots/part3_spot_pricing.png`
- `screenshots/part3_spot_request.png`
- `screenshots/part3_spot_preemption.png`
- `screenshots/part4_autoscaler_policy.png`
- `screenshots/part4_autoscaler_evaluation.png`
- `screenshots/part5_cost_snapshots.png`
- `screenshots/part5_waste_report.png`
- `screenshots/part5_recommendations.png`
- `screenshots/part5_dashboard.png`
- `screenshots/part6_cost_breakdown_viz.png`
- `screenshots/part6_timeseries_viz.png`
- `screenshots/part7_full_workflow.png`
- `screenshots/part8_gpu_detection.png`
- `screenshots/part8_gpu_metrics_diagnostic.png`
- `screenshots/part8_fp32_summary.png`
- `screenshots/part8_amp_summary.png`
- `screenshots/part8_fp32_vs_amp_comparison.png`
- `screenshots/part8_real_gpu_cost_report.png`
- `screenshots/part85_multi_gpu_analysis.png`
- `screenshots/part85_project_forecast.png`
- `screenshots/part85_optimization_analysis.png`
- `screenshots/part85_integrated_dashboard.png`
- `screenshots/part85_challange_strategy.png`

**Generated charts**
- `generated_charts/finops_cost_breakdown.png`
- `generated_charts/finops_timeseries.png`
- `generated_charts/real_gpu_comparison.png`
- `generated_charts/real_gpu_telemetry.png`
- `generated_charts/cost_per_epoch.png`
- `generated_charts/advanced_finops_dashboard.png`
