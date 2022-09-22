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
    stage('Build ISO') {
      steps {      
        sh '''xorriso -as mkisofs -V 'Linkat 22.04.1 LTS amd64' --grub2-mbr "${ISO_BASE}.mbr" -iso-level 3 --protective-msdos-label -partition_cyl_align off -partition_offset 16 --mbr-force-bootable -append_partition 2 28732ac11ff8d211ba4b00a0c93ec93b "${ISO_BASE}.efi" -appended_part_as_gpt -c '/boot.catalog' -b 'iso/boot/grub/i386-pc/eltorito.img' -no-emul-boot -boot-load-size 4 -boot-info-table --grub2-boot-info -e '--interval:appended_partition_2_start_1866280s_size_8496d:all::' -o "${ISO_LINKAT}.iso" /var/jenkins_home/workspace/iso-builder/newiso/'''
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
