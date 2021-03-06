# 1. Installation
### Install via composer
`composer require gfs/notifications ~v1.0`

### Add to AppKernel.php
```php
new GFS\NotificationBundle\NotificationBundle(),
```

# 2. Create Notification
```php
use GFS\NotificationBundle\Entity\Notification as Base;
class Notifications extends Base
{
    /**
    * @var int
    *
    * @ORM\Column(name="id", type="integer")
    * @ORM\Id
    * @ORM\GeneratedValue(strategy="AUTO")
    */
    private $id;

  /**
   * @var User
   *
   * @ORM\ManyToOne(targetEntity="User",inversedBy="notifications")
   * @ORM\JoinColumn(name="user_id", onDelete="cascade")
   */
  private $user;

  /**
   * @param UserInterface $user
   *
   * @return $this
   */
  public function setUser(UserInterface $user)
  {
      $this->user = $user;

      return $this;
  }

  /**
   * @return User
   */
  public function getUser()
  {
      return $this->user;
  }

  /**
   * Get id
   *
   * @return int
   */
  public function getId()
  {
      return $this->id;
  }

  /**
   * This is important because the server will send your JSON notifications ( json_encode(your notification) ).
   * You can customize what field you want the server to send to your client.
   */
  public function jsonSerialize()
  {
      return [
          'type' => $this->getType(),
          'description' => $this->getDescription(),
          'checked' => $this->getChecked(),
          'checkedAt' => $this->getCheckedAt(),
          'createdAt' => $this->getCreateAt(),
          'url' => $this->url,
          'id' => $this->id,
          'userId' => $this->user->getUsername() //This helps the server identify if a specific user is connected and only send notifications to that user. You can use another field for common notifications, for example group notifications.
      ];
  }
}
```

Function jsonSerialize should return an array that must contain the field 'userId', the server checks all connections if they contain this value.

# 3. Start server
Symfony 2 `php app/console server:notification`

Symfony 3 `php bin/console server:notification`

# 4. Client job
Use any WebSocket you want or any technologies. The most important thing is the URL, it must have GET parameter `userId`. That parameter will be bind to current connection.
This `userId` is used when you create a notification, the parameter `userId` comes from function `jsonSerialize`.
The GET parameter `userId` should be identical with the one returned by the function jsonSerialize, so the user receives the notification.

```javascript
var conn = new WebSocket('ws://yourip:8080?userId='+$scope.username);
conn.onopen = function(e) {
    console.log("Connection established!");
};

conn.onmessage = function(e) {
    var noitification = JSON.parse(e.data);
    // handle your notification object here
};
```

# 5. Create a notification
Create an entity: `Notification`. After successfully inserting it in the database, a notification will be sent to the server and the server will find the connection that matches `userId` field from `jsonSerialize`.

```php
 $notification = new Notifications();
 $notification->setType('type')
     ->setDescription('Your Notification here')
     ->setUser($user)
 ;
 $this->get('doctrine.orm.default_entity_manager')->persist($notification);
 $this->get('doctrine.orm.default_entity_manager')->flush();
```
# Configuration
```yaml
#config.yml
gfs_notifications:
    host: localhost # ip or DNS where server run default is localhost
    port: 8080 # port when server want run default is 8080
    notification: GFS\NotificationBundle\Notification\Notification # you can implement your own Ratchet\MessageComponentInterface
```

# Default Entity field:
- id
- type ( string, 255 )
- description ( text )
- created_at ( DateTime )
- checked_at ( DateTime, default null )
- checked ( boolean )
- user ( instance of Symfony\Component\Security\Core\User\UserInterface )

Remeber if you want rewrite __contruct don't forget to write:

`$this->created_at = new \DateTime('now')`.