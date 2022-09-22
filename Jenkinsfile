pipeline {
  agent any
  environment {
    ISO_BASE = 'ubuntu-22.04.1-desktop-amd64'
    ISO_LINKAT = 'linkat-22.04.1-desktop-amd64'
    
  }
  parameters {
    string(name: 'LOCALE', defaultValue: 'es_ES', description: 'Locale')
  	string(name: 'FULLNAME', defaultValue: 'Ubuntu', description: 'Full Name')
  	string(name: 'USERNAME', defaultValue: 'ubuntu', description: 'Username')
    string(name: 'PUBKEY', defaultValue: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCw73YVw5JKvNvATa7EAh/bQFDV41GKj2B2/bZ5QF+qJQXb08o1az0A8dsGbxlkkXPALxzmfWKcLxoDIOYD58kkPK/+eCbE+EDi/trpQ1cdltVvlC31cwfctlCOrdKboKjwqqUKsurfJY8zFlsYBras5IxdLSk/4VxOkC6O+/3+ptw+UAY5RGdAuwDppP60qi807t7ySPmtVx90I+I31rpzizzqfI/wkUutVonZKYn9A9nsF3Xkf2MQwtbA7OWfVv/0IkXdsTatQAFqkUPhO8ZOiMhCeb4E2juO/b0jCNxyieXfkxAkpONfLM0E+0HLvDsOjWE7mvcpuZ0hFDPFYk/d jenkins@jenkins', description: 'Public SSH key that will be added to authorized_keys for this user')
    string(name: 'NETIFACE', defaultValue: 'auto', description: 'Default network interface')
    string(name: 'HOSTNAME', defaultValue: 'unassigned-hostname', description: 'Hostname')
    string(name: 'DOMAIN', defaultValue: 'unassigned-domain', description: 'Domain')
    string(name: 'TIMEZONE', defaultValue: 'US/Eastern', description: 'Timezone')
    string(name: 'ROOT_DEV', defaultValue: '/dev/sda', description: 'System disk')
  }
  options {
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '3', daysToKeepStr: '', numToKeepStr: '3'))
  }
  stages {
    stage('Download ISO') {
      when {
        expression {
          return !(fileExists("${ISO_BASE}.iso"))
        }
        
      }
      steps {
        sh "curl -O https://releases.ubuntu.com/22.04/${ISO_BASE}.iso"
      }
    }
    stage('Mount ISO') {
      steps {                  
        sh '7z x ./${ISO_BASE}.iso -oiso'
      }
    }
    stage('Extract MBR partition image from the original ISO.') {
      steps {
          sh 'dd if="${ISO_BASE}.iso" bs=1 count=446 of="${ISO_BASE}.mbr"'
      }    
    }
    stage('Extract EFI partition image from the original ISO.') {
      steps {
        sh '''
        SKIP=$(/sbin/fdisk -l "${ISO_BASE}.iso" | fgrep '.iso2 ' | awk '{print $2}')
        SIZE=$(/sbin/fdisk -l "${ISO_BASE}.iso" | fgrep '.iso2 ' | awk '{print $4}')
        dd if="${ISO_BASE}.iso" bs=512 skip="$SKIP" count="$SIZE" of="${ISO_BASE}.efi"
        '''
        }    
    }
    stage('Configure') {
      steps {
        sh 'echo en > ./iso/isolinux/lang'
        sh 'cp ./preseed/server.seed ./iso/preseed'        
        sh '''
        PASSWORD=$(openssl rand -base64 32)
        echo "Password for ${USERNAME} is: \'${PASSWORD}\', make sure to change it on first boot"
        PASSWORD_CRYPT=$(mkpasswd -m sha-512 -S $(pwgen -ns 16 1) ${PASSWORD})
        PASSWORD_CRYPT_ESCAPED=$(echo "$PASSWORD_CRYPT" | sed \'s/[&/\\]/\\\\&/g\')
        sed -i "s/PASSWORD_CRYPT/${PASSWORD_CRYPT_ESCAPED}/g" ./iso/preseed/server.seed
        '''                  
      	sh 'sed -i -r "s#timeout\\s+[0-9]+#timeout 10#g" ./iso/isolinux/isolinux.cfg'        
        sh 'sed -i "s#NETIFACE#${NETIFACE}#g" ./iso/preseed/server.seed'
        sh 'sed -i "s#LOCALE#${LOCALE}#g" ./iso/preseed/server.seed'
        sh 'sed -i "s/FULLNAME/${FULLNAME}/g" ./iso/preseed/server.seed'
        sh 'sed -i "s/USERNAME/${USERNAME}/g" ./iso/preseed/server.seed'        
        sh 'sed -i "s/HOSTNAME/${HOSTNAME}/g" ./iso/preseed/server.seed'
        sh 'sed -i "s/DOMAIN/${DOMAIN}/g" ./iso/preseed/server.seed'
        sh 'sed -i "s#ROOT_DEV#${ROOT_DEV}#g" ./iso/preseed/server.seed'
        sh 'sed -i "s#TIMEZONE#${TIMEZONE}#g" ./iso/preseed/server.seed'
        sh '''        	
			MD5_SUM=$(md5sum ./iso/preseed/server.seed)
			sed -i "/label install/ilabel autoinstall\\n\\
			  menu label ^Autoinstall Ubuntu 16.04 Server\\n\\
			  kernel /install/vmlinuz\\n\\
			  append file=/cdrom/preseed/ubuntu-server.seed initrd=/install/initrd.gz auto=true priority=high preseed/file=/cdrom/preseed/server.seed preseed/file/checksum=$MD5_SUM --" ./iso/isolinux/txt.cfg
        '''
      }
    }
    stage('Build ISO') {
      steps {      
        sh 'xorriso -as mkisofs -r -V 'Linkat_2204_LTS_Desktop_EFIBIOS' -o /opt/ubnt/ubuntu-modif.iso --grub2-mbr /opt/ubnt/boot_hybrid.img  -iso-level 3 -partition_offset 16 --mbr-force-bootable -append_partition 2 28732ac11ff8d211ba4b00a0c93ec93b /opt/ubnt/efi.img -appended_part_as_gpt -iso_mbr_part_type a2a0d0ebe5b9334487c068b6b72699c7 -c '/boot.catalog' -b 'iso/boot/grub/i386-pc/eltorito.img' -no-emul-boot -boot-load-size 4 -boot-info-table --grub2-boot-info -eltorito-alt-boot -e '--interval\\:appended_partition_2\\:\\:\\:' -no-emul-boot /var/jenkins_home/workspace/iso-builder/'
      }
      post {
        success {          
          archiveArtifacts artifacts: "${ISO_BASE}_unattend.iso", fingerprint: true
        }
      }
    }   
  }
  post {  
	  always {
	    dir('iso') {
        deleteDir()
      }
      dir('initrd') {
        deleteDir()
      }
	  }
	}   
}
