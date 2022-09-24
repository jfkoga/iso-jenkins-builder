pipeline {
  agent any
  environment {
    ISO_BASE = 'ubuntu-22.04.1-desktop-amd64'
    ISO_LINKAT = 'linkat-22.04.1-desktop-amd64'
    
  }
  parameters {
    string(name: 'LOCALE', defaultValue: 'ca_ES', description: 'Locale')
  	string(name: 'FULLNAME', defaultValue: 'Linkat Desktop', description: 'Full Name')
  	string(name: 'USERNAME', defaultValue: 'suport', description: 'Username')
    string(name: 'PUBKEY', defaultValue: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCw73YVw5JKvNvATa7EAh/bQFDV41GKj2B2/bZ5QF+qJQXb08o1az0A8dsGbxlkkXPALxzmfWKcLxoDIOYD58kkPK/+eCbE+EDi/trpQ1cdltVvlC31cwfctlCOrdKboKjwqqUKsurfJY8zFlsYBras5IxdLSk/4VxOkC6O+/3+ptw+UAY5RGdAuwDppP60qi807t7ySPmtVx90I+I31rpzizzqfI/wkUutVonZKYn9A9nsF3Xkf2MQwtbA7OWfVv/0IkXdsTatQAFqkUPhO8ZOiMhCeb4E2juO/b0jCNxyieXfkxAkpONfLM0E+0HLvDsOjWE7mvcpuZ0hFDPFYk/d jenkins@jenkins', description: 'Public SSH key that will be added to authorized_keys for this user')
    string(name: 'NETIFACE', defaultValue: 'auto', description: 'Default network interface')
    string(name: 'HOSTNAME', defaultValue: 'linkat-desktop', description: 'Hostname')
    string(name: 'DOMAIN', defaultValue: 'unassigned-domain', description: 'Domain')
    string(name: 'TIMEZONE', defaultValue: 'Europe/Barcelona', description: 'Timezone')
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
        //sh 'xorriso -osirrox on -indev "${ISO_BASE}.iso" -extract / iso && chmod -R +w iso'
        sh '7z x ./${ISO_BASE}.iso -oiso'
      }
    }
    stage('Extract MBR partition image from the original ISO.') {
      steps {
          sh 'dd if="${ISO_BASE}.iso" bs=1 count=432 of="${ISO_BASE}_mbr.img"'
      }    
    }
    stage('Extract EFI partition image from the original ISO.') {
      steps {
        //sh '''
        //SKIP=$(/sbin/fdisk -l "${ISO_BASE}.iso" | fgrep '.iso2 ' | awk '{print $2}')
        //SIZE=$(/sbin/fdisk -l "${ISO_BASE}.iso" | fgrep '.iso2 ' | awk '{print $4}')
        //dd if="${ISO_BASE}.iso" bs=512 skip="$SKIP" count="$SIZE" of="${ISO_BASE}_efi.img"
        //'''
        sh 'dd if="${ISO_BASE}.iso" bs=512 skip=7129428 count=8496 of="${ISO_BASE}_efi.img"'
        }    
    }
//    stage('Squashfs') {
//      steps {      
//        sh 'cp iso/casper/filesystem.squashfs .'
//        sh 'sudo unsquashfs filesystem.squashfs'
//      }
//
//    }       
    stage('Build ISO') {
      steps {      
        //sh '''xorriso -as mkisofs -r -V 'Linkat 22.04 LTS Desktop' -o ${ISO_LINKAT}.iso --grub2-mbr ${ISO_BASE}_mbr.img  -iso-level 3 -partition_offset 16 --mbr-force-bootable -append_partition 2 28732ac11ff8d211ba4b00a0c93ec93b ${ISO_BASE}_efi.img -appended_part_as_gpt -iso_mbr_part_type a2a0d0ebe5b9334487c068b6b72699c7   -c '/boot.catalog'  -b 'iso/boot/grub/i386-pc/eltorito.img'     -no-emul-boot -boot-load-size 4 -boot-info-table --grub2-boot-info   -eltorito-alt-boot   -e '--interval:appended_partition_2:::'     -no-emul-boot /var/jenkins_home/workspace/iso-builder/'''
        sh '''xorriso -as mkisofs -r -V 'Linkat 22.04 LTS Desktop' -o ${ISO_LINKAT}.iso --grub2-mbr ${ISO_BASE}_mbr.img  --protective-msdos-label -partition_cyl_align off -partition_offset 16 -iso-level 3 --mbr-force-bootable -append_partition 2 28732ac11ff8d211ba4b00a0c93ec93b ${ISO_BASE}_efi.img -appended_part_as_gpt -iso_mbr_part_type a2a0d0ebe5b9334487c068b6b72699c7   -c '/boot.catalog'  -b 'iso/boot/grub/i386-pc/eltorito.img'     -no-emul-boot -boot-load-size 4 -boot-info-table --grub2-boot-info -boot-load-size 8496 -isohybrid-gpt-basdat -eltorito-alt-boot -e '--interval:appended_partition_2:::' -no-emul-boot /var/jenkins_home/workspace/iso-builder/'''
      }
      post {
        success {          
          archiveArtifacts artifacts: "${ISO_LINKAT}.iso", fingerprint: true
        }
      }
    }   
  }
//  post {  
//	  always {
//	    dir('iso') {
//        deleteDir()
//      }
//	  }
//	}   
}
