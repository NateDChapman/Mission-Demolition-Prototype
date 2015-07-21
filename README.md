# Mission-Demolition-Prototype
Mission Demolition Prototype for Prototyping Class

-----------------------------
Goal Script

using UnityEngine;
using System.Collections;

public class Goal : MonoBehaviour {

	static public bool goalMet = false;

	void OnTriggerEnter ( Collider other ) {
		if (other.gameObject.tag == "Projectile") {
			Goal.goalMet = true;
			Color c = GetComponent<Renderer> ().material.color;
			c.a = 1;
			GetComponent<Renderer> ().material.color = c;
		}
	}
}
------------------------------------
Follow Cam Script

sing UnityEngine;
using System.Collections;

public class FollowCam : MonoBehaviour {

	static public FollowCam S;

	public float easing = 0.05f;
	public Vector2 minXY;
	public bool _________;
	public GameObject poi;
	public float camZ;

	void Awake() {
		S = this;
		camZ = this.transform.position.z;
	}
	
	void FixedUpdate () {
		Vector3 destination;
		if (poi == null) {
			destination = Vector3.zero;
		} else {
			destination = poi.transform.position;
			if (poi.tag == "Projectile") {
				if (poi.GetComponent<Rigidbody>().IsSleeping ()) {
					poi = null;
					return;
				}
			}
		}
		destination.x = Mathf.Max (minXY.x, destination.x);
		destination.y = Mathf.Max (minXY.y, destination.y);
		destination = Vector3.Lerp (transform.position, destination, easing);
		destination.z = camZ;
		transform.position = destination;
		this.GetComponent<Camera>().orthographicSize = destination.y + 10;
	}
}
----------------------------
Slingshot Script

using UnityEngine;
using System.Collections;

public class Slingshot : MonoBehaviour {

	public GameObject prefabProjectile;
	public float velocityMult = 4f;
	public bool ________;
	public GameObject launchPoint;
	public Vector3 launchPos;
	public GameObject projectile;
	public bool aimingMode;

	void Awake() {
		Transform launchPointTrans = transform.Find ("LaunchPoint");
		launchPoint = launchPointTrans.gameObject;
		launchPoint.SetActive (false);
		launchPos = launchPointTrans.position;
	}
	void OnMouseEnter() {
		launchPoint.SetActive (true);
	}

	void OnMouseExit() {
		launchPoint.SetActive (false);
	}

	void OnMouseDown() {
		aimingMode = true;
		projectile = Instantiate (prefabProjectile) as GameObject;
		projectile.transform.position = launchPos;
		projectile.GetComponent<Rigidbody>().isKinematic = true;
	}

	void Update() {
		if (!aimingMode) return;
		Vector3 mousePos2D = Input.mousePosition;
		mousePos2D.z = -Camera.main.transform.position.z;
		Vector3 mousePos3D = Camera.main.ScreenToWorldPoint (mousePos2D);
		Vector3 mouseDelta = mousePos3D - launchPos;
		float maxMagnitude = this.GetComponent<SphereCollider> ().radius;
		if (mouseDelta.magnitude > maxMagnitude) {
			mouseDelta.Normalize ();
			mouseDelta *= maxMagnitude;
		}

		Vector3 projPos = launchPos + mouseDelta;
		projectile.transform.position = projPos;
		if (Input.GetMouseButtonUp (0)) {
			aimingMode = false;
			projectile.GetComponent<Rigidbody> ().isKinematic = false;
			projectile.GetComponent<Rigidbody> ().velocity = -mouseDelta * velocityMult;
			FollowCam.S.poi = projectile;
			projectile = null;
		}
	}
}
------------------------
Cloud Crafter Script

using UnityEngine;
using System.Collections;

public class CloudCrafter : MonoBehaviour {

	public int numClouds = 40;
	public GameObject[] cloudPrefabs;
	public Vector3 cloudPosMin;
	public Vector3 cloudPosMax;
	public float cloudScaleMin = 1;
	public float cloudScaleMax = 5;
	public float cloudSpeedMult = 0.5f;

	public bool ________;

	public GameObject[] cloudInstances;

	void Awake() {
		cloudInstances = new GameObject[numClouds];
		GameObject anchor = GameObject.Find ("CloudAnchor");
		GameObject cloud;
		for (int i=0; i<numClouds; i++) {
			int prefabNum = Random.Range (0, cloudPrefabs.Length);
			cloud = Instantiate (cloudPrefabs [prefabNum]) as GameObject;
			Vector3 cPos = Vector3.zero;
			cPos.x = Random.Range (cloudPosMin.x, cloudPosMax.x);
			cPos.y = Random.Range (cloudPosMin.y, cloudPosMax.y);
			float scaleU = Random.value;
			float scaleVal = Mathf.Lerp (cloudScaleMin, cloudScaleMax, scaleU);
			cPos.y = Mathf.Lerp (cloudPosMin.y, cPos.y, scaleU);
			cPos.z = 100 - 90 * scaleU;
			cloud.transform.position = cPos;
			cloud.transform.localScale = Vector3.one * scaleVal;
			cloud.transform.parent = anchor.transform;
			cloudInstances [i] = cloud;
		}	
	}
	void Update() {
		foreach (GameObject cloud in cloudInstances) {
			float scaleVal = cloud.transform.localScale.x;
			Vector3 cPos = cloud.transform.position;
			cPos.x -= scaleVal * Time.deltaTime * cloudSpeedMult;
			if (cPos.x <= cloudPosMin.x) {
				cPos.x = cloudPosMax.x;
			}
			cloud.transform.position = cPos;
		}
	}
}
-----------------------------------------
