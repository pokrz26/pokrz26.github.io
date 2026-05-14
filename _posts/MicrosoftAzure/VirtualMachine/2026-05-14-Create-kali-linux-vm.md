Create kali linux vm

--image Publisher:Offer:Sku:Version

"imageReference": {
    "publisher": "kali-linux",
    "offer": "kali",
    "sku": "kali-2026-1",
    "version": "latest"
}

--image kali-linux:kali:kali-2026-1:latest

-------- ???????
az vm create `
  --resource-group team-red-burp-rg-test `
  --name team-red-burp-1-vm-test `
  --size Standard_D8as_v6 `
  --image kali-linux:kali:kali-2026-1:latest `
  --plan-name kali-2026-1 `
  --plan-product kali `
  --plan-publisher kali-linux `
  --admin-username insadmin `
  --generate-ssh-keys `
  --os-disk-size-gb 128 `
  --os-disk-delete-option Delete `
  --storage-sku StandardSSD_LRS `
  --public-ip-sku Standard `
  --nsg team-red-burp-vm-test-nsg

az vm auto-shutdown `
  -g team-red-burp-rg-test `
  -n team-red-burp-0-vm-test `
  --time 1800
-------- ???????

login with ssh -> ssh -i .\inadmin.pem insadmin@4.180.148.190