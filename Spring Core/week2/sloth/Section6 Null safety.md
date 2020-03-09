# Section6 Null-safety

툴의 지원을 받아서 컴파일 타임에 NPE를 최대한 미연에 방지하는 것.

    public class EventService {
    	
    	@NonNull // return null 허용x  
    	public String createEvnet(@NonNull String name) {
    		return "hello" + name
    	}
    
    }
    
    
    public class DareunClass { 
    
    	EventService eventService = new EventService();
    	eventService.crateEvent(null);
    
    }

@NonNull 애노테이션을 여기저기 덕지덕지 붙여 놓으면 안 좋지 않나..?