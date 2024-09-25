Yii EAuth extension
===================

���������� EAuth ��������� �������� �� ���� ����������� � ������� OpenID � OAuth �����������.

�������������� ���������� "�� �������":

* OpenID: Google, ������;
* OAuth: Twitter;
* OAuth 2.0: Google, Facebook, ���������, Mail.ru.


### ������

* [Yii EAuth](https://github.com/Nodge/yii-eauth)
* [Yii Framework](http://yiiframework.com/)
* [OpenID](http://openid.net/)
* [OAuth](http://oauth.net/)
* [OAuth 2.0](http://oauth.net/2/)
* [loid extension](http://www.yiiframework.com/extension/loid)
* [EOAuth extension](http://www.yiiframework.com/extension/eoauth)


### ��������� ����������

* Yii 1.1 or above
* PHP curl extension
* [loid extension](http://www.yiiframework.com/extension/loid)
* [EOAuth extension](http://www.yiiframework.com/extension/eoauth)


## ���������

* ���������� ���������� loid � EOAuth.
* ����������� ���������� EAuth � ���������� `protected/extensions`.
* �������� ��������� ������ � ���� ������������ `protected/config/main.php`:

```php
<?php
...
	'import'=>array(
		'ext.eoauth.*',
		'ext.eoauth.lib.*',
		'ext.lightopenid.*',
		'ext.eauth.services.*',
	),
...
	'components'=>array(
		'loid' => array(
			'class' => 'ext.lightopenid.loid',
		),
		'eauth' => array(
			'class' => 'ext.eauth.EAuth',
			'popup' => true, // ������������ ����������� ���� ������ ��������������� �� ���� ����������
			'services' => array( // �� ������ ��������� ������ ����������� � �������������� �� ������
				'google' => array(
					'class' => 'GoogleOpenIDService',
				),
				'yandex' => array(
					'class' => 'YandexOpenIDService',
				),
				'twitter' => array(
					'class' => 'TwitterOAuthService',
					'key' => '...',
					'secret' => '...',
				),
				'facebook' => array(
					'class' => 'FacebookOAuthService',
					'client_id' => '...',
					'client_secret' => '...',
				),
				'vkontakte' => array(
					'class' => 'VKontakteOAuthService',
					'client_id' => '...',
					'client_secret' => '...',
				),
				'mailru' => array(
					'class' => 'MailruOAuthService',
					'client_id' => '...',
					'client_secret' => '...',
				),
			),
		),
		...
	),
...
```


## �������������

#### ����� UserIdentity

```php
<?php

class ServiceUserIdentity extends UserIdentity {
	const ERROR_NOT_AUTHENTICATED = 3;

	/**
	 * @var EAuthServiceBase the authorization service instance.
	 */
	protected $service;
	
	/**
	 * Constructor.
	 * @param EAuthServiceBase $service the authorization service instance.
	 */
	public function __construct($service) {
		$this->service = $service;
	}
	
	/**
	 * Authenticates a user based on {@link username}.
	 * This method is required by {@link IUserIdentity}.
	 * @return boolean whether authentication succeeds.
	 */
	public function authenticate() {		
		if ($this->service->isAuthenticated) {
			$this->username = $this->service->getAttribute('name');
			$this->setState('id', $this->service->id);
			$this->setState('name', $this->username);
			$this->setState('service', $this->service->serviceName);
			$this->errorCode = self::ERROR_NONE;		
		}
		else {
			$this->errorCode = self::ERROR_NOT_AUTHENTICATED;
		}
		return !$this->errorCode;
	}
}
```

#### �������� � �����������

```php
<?php
...
	public function actionLogin() {
		$service = Yii::app()->request->getQuery('service');
		if (isset($service) in_array($service, array())) {
			$authIdentity = Yii::app()->eauth->getIdentity($service);
			$authIdentity->redirectUrl = Yii::app()->user->returnUrl;
			$authIdentity->cancelUrl = $this->createAbsoluteUrl('site/login');
			
			if ($authIdentity->authenticate()) {
				$identity = new ServiceUserIdentity($authIdentity);
				
				// �������� �����������
				if ($identity->authenticate()) {
					Yii::app()->user->login($identity);
					
					// ����������� ��������������� ��� ����������� �������� ������������ ����
					$authIdentity->redirect();
				}
				else {
					// �������� ������������ ���� � ��������������� �� cancelUrl
					$authIdentity->cancel();
				}
			}
			
			// ����������� �� �������, �������������� �� �������� �����
			$this->redirect(array('site/login'));
		}
		
		// ����� ����������� ��� ����������� �� ������/������...
	}
```

#### �������������

```php
<h2>������� �� ������ ��� ����� ����� ���� �� ������:</h2>
<?php 
	Yii::app()->eauth->renderWidget();
?>
```


## ��������

��������� ����� ����� � ���������� ������ ���������� ��� ������ ������� [LiStick.ru](http://listick.ru). �� ������ ������ � ��������� ������������ ����������.

���������� ��������������� ��� ��������� [New BSD License](http://www.opensource.org/licenses/bsd-license.php), ��� ��� ��������� ������ ����� ����� �� [GitHub](https://github.com/Nodge/yii-eauth).
