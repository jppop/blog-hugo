+++
title = "good to great"
draft = false
date = "2017-04-08T11:09:35+02:00"
disable_profile = true
disable_widgets = false
tags = ["test"]
+++

Hugo test

# heading 1

Lorem ipsum dolor sit amet, consectetur adipiscing elit. In auctor leo at hendrerit porta. Ut nec pulvinar augue. Mauris vestibulum enim velit, venenatis dapibus odio euismod non. In eget maximus tellus. Sed ipsum arcu, varius eu pharetra nec, volutpat nec elit. Suspendisse sed cursus tellus. Phasellus justo lectus, euismod ac lectus vitae, aliquet pulvinar nibh. Vivamus semper urna efficitur risus efficitur dapibus. Suspendisse quis suscipit risus. Quisque libero sapien, sodales ultrices fermentum sit amet, facilisis non arcu. Nulla mattis tempus urna, vitae tristique enim facilisis et. Ut consequat scelerisque neque, non lobortis lorem placerat ut. Donec eros dolor, maximus nec semper id, euismod in est. Aenean venenatis et est fermentum facilisis.

```bash
cd here
ls -al
```
Proin vel odio eu sapien efficitur mattis ut a ipsum. Integer in massa imperdiet, sodales lectus mollis, fermentum nulla. Cras aliquet sem vitae magna accumsan, non feugiat eros suscipit. Sed vitae vehicula arcu. In hac habitasse platea dictumst. Sed a feugiat sem. Sed ex mi, consectetur sed magna a, accumsan scelerisque ligula. Duis elementum sapien purus, sit amet consequat ipsum aliquam id. Morbi rhoncus lobortis mollis. Donec pharetra, metus at posuere auctor, quam justo tincidunt turpis, vitae iaculis ante quam vitae massa. Aliquam volutpat malesuada diam quis vulputate. Maecenas maximus mi dignissim dignissim feugiat. Vestibulum vestibulum congue dapibus. Phasellus sit amet nisl pulvinar, dapibus mi vitae, posuere erat. Fusce sapien mauris, cursus nec scelerisque a, vestibulum vitae neque. Phasellus nunc odio, volutpat at nisl id, cursus aliquam sem.

```Java
package org.pockito.xcp.message;

import java.text.MessageFormat;
import java.util.MissingResourceException;
import java.util.ResourceBundle;

public enum Message {
	E_OOPS_ERROR,
	E_ENTITYMGR_CREATION_FAILED, E_REMOVE_FAILED, E_NO_ID_KEY, E_FIND_FAILED, E_PERSIST_FAILED, E_NOT_PRIMITIVE_TYPE,
	E_ILLEGAL_RELATION_USE, E_REPO_OBJ_NOT_FOUND, E_ADD_ATTACHMENT_FAILED, E_CONTENTLESS_OBJ, E_GET_ATTACHMENT_FAILED,
	E_PARENT_MISSING, E_CHILD_MISSING, E_UNKNOWN_FIELD, E_NOT_SELECT_QUERY, E_CONFIRM_DELETE_ALL,
	E_NOT_SUPPORTED, E_NO_ENTITY, E_NO_ATTRIBUTE_ID, E_NO_ENTITY_ANNOTATION, E_NO_TYPE_ANNOTATION,
	E_NO_ID_ANNOTATION, E_NO_JAVABEAN, E_REFLECTION_FIELD, E_NO_SETTER, E_COLLECTION_FIELD,

	E_DFC_SESSMGR_FAILED, E_DFC_SESSION_FAILED, E_DFC_QUERY_FAILED, E_DFC_NO_SESSIONMGR, E_TX_ALREADY_STARTED,
	E_TX_START_FAILED, E_TX_COMMIT_FAILED, E_TX_NOT_ACTIVE, E_TX_CANNNOT_COMMIT, E_TX_ROLLBACK_FAILED,

	E_NO_REPOSITORY_CONTEXT,
	;

	public static final String COMPONENT_CODE = "XDP"; // Xcp Data Persistence

	private static final ResourceBundle resource = Utf8ResourceBundle.getBundle(Message.class.getName());

	public String get() {
		return get(resource, this, null);
	}

	public String get(Object... args) {
		return getMessage(resource, this, args);
	}

	public static String getMessage(ResourceBundle resource, @SuppressWarnings("rawtypes") Enum messageCode,
			Object... args) {
		StringBuffer buffer = new StringBuffer();
		buffer.append("[").append(COMPONENT_CODE).append(String.format("%04d", messageCode.ordinal())).append("] ")
			.append(messageCode.toString()).append(" - ");
		try {
			final String format = resource.getString(messageCode.toString());
			final String msg = MessageFormat.format(format, args);
			buffer.append(msg);
		} catch (MissingResourceException e) {
			buffer.append("!! message not found in the resource bundle !!");
		}
		return buffer.toString();
	}

}
```

Nam malesuada quam elementum turpis semper rhoncus. Ut mollis faucibus ante, eget tincidunt augue condimentum sit amet. Pellentesque mattis leo ligula, nec suscipit ipsum viverra at. Aenean vestibulum ex id eros dictum sollicitudin. Aliquam a orci mattis, semper odio eu, posuere mauris. Mauris hendrerit augue nec tincidunt suscipit. In condimentum metus in diam sagittis, sed semper justo ornare. Pellentesque accumsan elit id sem laoreet, id condimentum lectus scelerisque. Sed vehicula, felis semper consectetur euismod, ex ex luctus sem, quis commodo quam lacus non orci. Curabitur lacinia metus a nibh hendrerit convallis. Suspendisse auctor elit ut est lacinia, nec aliquam felis faucibus. Proin ligula tortor, vulputate sed sem quis, porttitor blandit neque.

Phasellus gravida nisl eget lectus hendrerit, nec consectetur justo feugiat. Praesent sit amet volutpat lacus. Nam blandit sem id ante suscipit, vel ultricies nisi tempor. Aliquam elementum tellus eu tempor ultrices. Donec sodales nulla lectus, sit amet vulputate quam viverra a. Curabitur elit massa, fermentum id tristique in, hendrerit eu nunc. Morbi sit amet massa ac mauris facilisis accumsan non sed lorem. Maecenas neque lacus, vulputate quis mollis id, condimentum eu dui. Vestibulum auctor eleifend odio, vel finibus nisl dictum vehicula. Duis ullamcorper aliquet mauris sed dictum. Maecenas egestas quam ac purus gravida accumsan.

Nulla et maximus erat. Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos. Nullam pellentesque at enim id sagittis. Maecenas vitae fringilla metus. Donec placerat mauris est, sed ultricies enim eleifend et. Integer in accumsan eros, vel posuere purus. Cras quis orci facilisis, blandit arcu vel, aliquam mi.
