[快速开始](https://mariadb.com/kb/en/library/a-mariadb-primer/)  [MySQL GUI Tools](https://downloads.mysql.com/archives/gui/)  
连接到数据库：  
```
$ mysql -u //user_name// -p -h //ip_address// //db_name//
```
按提示输入密码，就进入了mysql的命令行。  
```
MySQL [od]> show tables;
+-------------------------------+
| Tables_in_od                  |
+-------------------------------+
| ctg_cuni_caict_transfer_ws    |
| ctg_cuni_caict_transfer_zzxk  |
| ctg_sif_caict_transfer_ym     |
| ctg_sif_icp_gn_baxx_slxx      |
| developer_info                |
| ex_catalog_visit_log          |
| ex_share_record               |
| ex_share_resource_sub         |
| int_dp_table                  |
| int_dp_table_his              |
| model_level                   |
| model_quota_rule              |
| model_quota_rule_all          |
| ms_dxsz_caict_transfer_ws     |
| ms_dxsz_caict_transfer_zzxk   |
| ms_ipc_icp_gn_baxx_slxx       |
| ms_ymsp_caict_transfer_ym     |
| od_apply_interflow            |
| od_data_catalog               |
| od_data_catalog_index         |
| od_data_comparison_column     |
| od_data_comparison_condition  |
| od_data_comparison_filter     |
| od_data_comparison_log        |
| od_data_comparison_log_cdt    |
| od_data_comparison_log_result |
| od_data_comparison_moudle     |
| od_data_comparison_table      |
| od_data_comparison_table_rlt  |
| od_data_correct_log           |
| od_data_dict                  |
| od_data_dict_catalog          |
| od_data_dict_item             |
| od_data_encrypt_key           |
| od_data_encrypt_log           |
| od_data_encrypt_type          |
| od_data_item                  |
| od_data_mask_log              |
| od_data_mask_rule             |
| od_data_mask_rule_settings    |
| od_data_quality_rule          |
| od_data_quality_rule_catalog  |
| od_data_quality_task          |
| od_data_quality_task_field    |
| od_data_quantity_statistics   |
| od_data_resource              |
| od_data_resource_detect_log   |
| od_data_resource_grade        |
| od_data_resource_img          |
| od_data_share                 |
| od_data_share_where           |
| od_data_source                |
| od_data_source_catalog        |
| od_data_tag                   |
| od_dataitems_apply            |
| od_dataresource_apply         |
| od_dict_stangdrd_source       |
| od_organ_params               |
| od_quality_check_total        |
| od_quality_check_unmatched    |
| od_resource_catalog_lk        |
| od_resource_rate_correct_deal |
| od_standard_test              |
| od_standardstore_origin       |
| od_system_setting             |
| od_system_setting_type        |
| od_table_structure_log        |
| od_tag_catalog                |
| od_tag_stangdrd_source        |
| od_xtzddy                     |
| od_xtzddy_dl                  |
| od_xtzddy_xt                  |
| require_cat                   |
| sc_deploy_apps                |
| sc_deploy_apps_instances      |
| sc_deploy_hosts               |
| sc_deploy_instances           |
| sc_dict                       |
| sc_dict_item                  |
| sc_params_def                 |
| sc_params_value               |
| t                             |
+-------------------------------+
```
显示表字段：  
```
> DESCRIBE <table>;
```